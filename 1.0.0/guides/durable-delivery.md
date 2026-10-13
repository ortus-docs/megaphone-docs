# Durable Delivery

Want to save a Notification now and let a worker send it later? Megaphone includes cbq and a delivery job for that. You use the same Notifications, Notifiables, and channels as `notify()`, then call `queue()` to hand the work to cbq.

### Setting up cbq

The default queue uses cbq's database Provider, the `default` connection, and the `notifications` queue. Start by registering your channels and telling Megaphone how to load a Notifiable in the worker:

```js
moduleSettings.megaphone = {
    "channels": {
        "database": {
            "provider": "DatabaseProvider@megaphone",
            "properties": {}
        }
    },
    "durable": {
        "notifiableResolver": "NotifiableResolver"
    }
};
```

`NotifiableResolver` is your application component. Its `resolve(type, id)` method returns the current Notifiable, or null if it no longer exists. Megaphone uses it to load the recipient when the job runs.

Apply Megaphone's migrations and cbq's migrations through your application's migration ledger before queueing. cbq's migrations are in its `resources/database/migrations` directory. Use the same datasource for the delivery records and cbq jobs so they commit together. Megaphone's migrations don't create cbq's tables.

If your application doesn't have a cbq configuration, Megaphone supplies a database connection using the delivery store's query options. With `notifiableResolver` configured, it also defines one worker for the `notifications` queue. Applications that only use `notify()` don't start that worker or need queue tables.

If you already configure cbq, Megaphone keeps your configuration. Use its database Provider on the delivery datasource, and add a worker for notifications. For example, inside your cbq config's `configure()` method:

```js
newWorkerPool( "notifications" )
    .forConnection( "default" )
    .onQueue( "notifications" )
    .quantity( 1 )
    .timeout( 60 )
    .maxAttempts( 1 );
```

Each job carries only the delivery ID. Megaphone loads the saved Notification and decides whether another job is needed after a known retryable or deferred outcome. cbq gets one attempt per job, and an ambiguous outcome never schedules another send.

The bundled adapter requires cbq's database Provider. It rejects other Providers, including the synchronous Provider, before publication can commit or transport can begin. Use cbq's `registerWorkers=false` setting on a process that only publishes jobs.

### Queueing a Notification

Once cbq is set up, queue a Notification just as you would call `notify()`:

```js
var publication = megaphone.queue(
    notifiables = user,
    notification = "StockRebalancingCompleteNotification",
    properties = {
        "stockSymbol": "APPL",
        "completionTimestamp": now()
    }
);
```

You can pass a Notification instance or its WireBox name, along with a struct of properties to populate. Megaphone calls `via()` for each Notifiable, checks their channel preferences, and saves the resulting deliveries. The bundled cbq job sends them through the registered Providers.

Megaphone saves the event, deliveries, and cbq jobs in the same datasource transaction. If inserting a job fails, it rolls back the publication too.

The returned `NotificationPublication` gives you the saved event and its deliveries:

```js
var eventId = publication.getEvent().getId();
var deliveries = publication.getDeliveries();
```

If you use the [SendsNotifications delegate](../reference/sendsnotifications.md), you can queue from the Notifiable too:

```js
user.queue(
    "StockRebalancingCompleteNotification",
    { "stockSymbol": "APPL", "completionTimestamp": now() }
);
```

### Planning before queueing

Need to capture a notification now and hand it to a worker later? Call `plan()` with the same arguments. It returns a `NotificationPlan` without saving deliveries or sending anything.

The plan captures who should receive the notification, which channels they should receive it on, and which push devices to use. Saving and restoring that plan keeps those original decisions. For example, opting into email later won't add an email delivery to an older plan.

See [Event Identities](event-identities.md) for saving plans, choosing stable event keys, and handling repeated requests.

### Preparing your Notification

Your Notification still extends [BaseNotification](../reference/basenotification.md) and implements `via()` and the matching `to{ChannelType}` methods.

Queued notifications also need enough saved data to recreate the Notification in a worker. If you pass a properties struct to `queue()`, `getQueueProperties()` returns those properties by default. If you build the Notification with setters, override `getQueueProperties()` to return the values you want to save:

```js
public struct function getQueueProperties() {
    return {
        "stockSymbol": getStockSymbol(),
        "completionTimestamp": getCompletionTimestamp()
    };
}
```

Keep these values JSON-safe. Save IDs and plain data; let the worker resolve users and construct mail or request objects when it sends the Notification.

You can override `getChannelPreferences(notifiable)` to apply your application's preferences. Channels returned by `via()` are enabled by default. A disabled channel is skipped by `notify()` and saved as suppressed by `queue()`. That suppressed delivery stays suppressed if the user changes their mind later.

### Configuring durable delivery

You can choose a cbq connection, queue name, and timeout in the `durable` settings. Match the connection and queue to your cbq worker configuration. The defaults are `default`, `notifications`, and 60 seconds.

As your application needs more channels or policies, you can add them to the same module settings:

```js
moduleSettings.megaphone = {
    "channels": {
        "database": {
            "provider": "DatabaseProvider@megaphone",
            "properties": {}
        },
        "email": {
            "provider": "EmailProvider@megaphone",
            "properties": { "durableAdapter": "NotificationMailAdapter" }
        },
        "push": {
            "provider": "WebPushProvider@megaphone",
            "properties": { "runtime": "ApplicationPushRuntime" }
        }
    },
    "storage": {
        "events": {},
        "deliveries": {},
        "preferences": {},
        "subscriptions": {}
    },
    "durable": {
        "queueConnection": "default",
        "queueName": "notifications",
        "queueTimeout": 60,
        "notifiableResolver": "NotifiableResolver",
        "eligibilityPolicy": "NotificationEligibility",
        "snapshotHasher": "NotificationSnapshotHasher@megaphone",
        "leaseSeconds": 60,
        "maxAttempts": 5,
        "retrySeconds": 30,
        "maxRecoveryAttempts": 5,
        "maxRecoveryAllowance": 45
    }
};
```

You can omit the queue settings to use their defaults. The resolver and eligibility mappings are your application components; `NotificationSnapshotHasher` is bundled. Here is what the application callbacks do:

| Setting | Method | What your application supplies |
| --- | --- | --- |
| `notifiableResolver` | `resolve(type, id)` | The current Notifiable, or null if it no longer exists. |
| `eligibilityPolicy` | `check(delivery, notifiable, event)` | A `DeliveryPreparation` describing whether this notification can still be sent. |
| `deliveryPolicy` | `getDispatchOptions(id)` | Optional lease, attempt, and retry settings for a delivery. |
| `snapshotHasher` | `digest(value, context)` | A stable hash when your application needs its own hashing rules. |

### Application policies

Use the eligibility policy for application rules that may have changed since planning, such as access to the underlying record or whether a feature is enabled. Return a `DeliveryPreparation` with a status of `ready`, `suppressed`, `deferred`, or `permanent`, and an optional reason and retry date. Megaphone also checks current channel preferences before sending.

The storage settings accept `table`, `queryOptions`, and `datasource`. Delivery storage also accepts `attemptsTable`, `eventsTable`, and `recoveriesTable`; subscription storage accepts `exclusionsTable`. If you customize table names, update your migrations and related table settings to match. Public APIs use `notifiableType`, `notifiableId`, and `channelName`; existing database column names stay the same.

### What the cbq worker does

The bundled cbq job passes its saved delivery ID to `NotificationDeliveryService`. It calls this for you:

```js
var result = notificationDeliveryService.dispatch( deliveryId );
var status = result.getStatus();
```

Megaphone reconstructs the Notification and Notifiable, checks eligibility, and asks the stored channel to prepare the delivery. It checks eligibility again and records the start of the transport before sending. You can pass an `options` struct to `dispatch()` to override lease, attempt, and retry settings; those values take precedence over the configured delivery policy.

Each delivery has a lease so two workers can't send it at the same time. Preparation failures and feature pauses don't use up transport attempts.

{% hint style="warning" %}
If a transport throws after sending starts, Megaphone may not know whether the Provider accepted the notification. It records an ambiguous outcome and does not automatically resend it. See [Delivery Operations](delivery-operations.md) for reconciling Provider receipts.
{% endhint %}

An accepted delivery means the Provider accepted it. It doesn't mean the user has received or read it. Keep Provider I/O outside application transactions.

### Supporting durable delivery in a Provider

Providers can support the optional `IDurableNotificationProvider` protocol with two methods:

| Method | Return | Purpose |
| --- | --- | --- |
| `prepareDelivery(notifiable, notification, delivery)` | `DeliveryPreparation` | Build and validate the content before making a transport request. |
| `sendDelivery(prepared, delivery)` | `DeliveryResult` | Send the prepared content and report the outcome. |

`DeliveryResult` supports `accepted`, `retryable`, `permanent`, and `ambiguous`, with optional reason, Provider reference, and retry date. Use `prepared.getData()` for temporary mail or request objects. The preparation memento contains status metadata, so those transport objects aren't serialized.

The Database and Web Push Providers support durable delivery. The Email Provider needs a configured durable adapter that supplies explicit evidence of acceptance. A successful method return alone isn't enough to establish acceptance.

You can keep using a Provider that only implements `notify()` for synchronous notifications. If you try to plan a durable notification for that channel, Megaphone throws `Megaphone.Configuration.UnsupportedDurableProvider`.

### Making a custom queue adapter

If your application needs a different job queue or extra work when inserting a job, you can supply your own adapter. It needs an `enqueue(delivery)` method that receives a `NotificationDelivery` object.

Here is an example using an application queue service:

```js
// models/NotificationQueueAdapter.cfc
component singleton {

    property name="jobQueue" inject="JobQueue";

    public void function enqueue( required any delivery ) {
        variables.jobQueue.enqueue(
            job = "DeliverNotification",
            properties = { "deliveryId": arguments.delivery.getId() },
            availableDate = arguments.delivery.getAvailableDate()
        );
    }

}
```

`JobQueue` and its `enqueue()` method in this example belong to your application. That method should insert the job in the same datasource transaction as publication. Let insertion failures propagate so Megaphone can roll back the event and deliveries too. The adapter queues the ID; sending happens later in the worker.

Register the adapter in your module settings:

```js
moduleSettings.megaphone.durable.queueAdapter = "NotificationQueueAdapter";
```

You can also supply an adapter instance. Without an override, Megaphone uses `CBQNotificationQueue@megaphone`.

Your worker calls `NotificationDeliveryService.dispatch(deliveryId)` and uses the returned `DeliveryResult`. If you schedule another job after a `retryable`, `deferred`, or `busy` result, reload the delivery and respect its current state and availability date. For a busy delivery, wait until its lease expires. Let Megaphone control the transport allowance, and never automatically resend an accepted or ambiguous delivery. See [Delivery Operations](delivery-operations.md) for recovery and receipt reconciliation.

If you prefer a scheduled polling worker, explicitly set `durable.queueAdapter` to an empty string. Megaphone still saves deliveries; your worker can find them with `getDueDeliveries()` and handle expired leases with `recoverExpired()`.
