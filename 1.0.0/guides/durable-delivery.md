# Durable Delivery

Sometimes you want to send a notification as part of a request. Other times you want to save it now and let a worker send it later. Megaphone supports both using the same Notifications, Notifiables, and channels you already use.

### Queueing a Notification

If you have used `notify()`, `queue()` should look familiar:

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

You can pass a Notification instance or its WireBox name, along with a struct of properties to populate. Megaphone calls `via()` for each Notifiable, checks their channel preferences, and saves the resulting deliveries. A worker can then send them through the registered Providers.

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

Configure the queue and application callbacks in your Megaphone module settings:

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
        "queueAdapter": "NotificationQueue",
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

The mappings in this example are application components, except for the bundled `NotificationSnapshotHasher`. Here is what each callback does:

| Setting | Method | What your application supplies |
| --- | --- | --- |
| `queueAdapter` | `enqueue(delivery)` | A job containing `delivery.getId()`. |
| `notifiableResolver` | `resolve(type, id)` | The current Notifiable, or null if it no longer exists. |
| `eligibilityPolicy` | `check(delivery, notifiable, event)` | A `DeliveryPreparation` describing whether this notification can still be sent. |
| `deliveryPolicy` | `getDispatchOptions(id)` | Optional lease, attempt, and retry settings for a delivery. |
| `snapshotHasher` | `digest(value, context)` | A stable hash when your application needs its own hashing rules. |

`queue()` saves the event, deliveries, and queue jobs in the same datasource transaction. Your queue adapter should only insert the job here. Sending happens in the worker. If inserting a job fails, Megaphone rolls back the publication too.

If you don't configure a queue adapter, you can schedule a worker that uses `NotificationDeliveryService.getDueDeliveries()` and `recoverExpired()`.

Use the eligibility policy for application rules that may have changed since planning, such as access to the underlying record or whether a feature is enabled. Return a `DeliveryPreparation` with a status of `ready`, `suppressed`, `deferred`, or `permanent`, and an optional reason and retry date. Megaphone also checks current channel preferences before sending.

The storage settings accept `table`, `queryOptions`, and `datasource`. Delivery storage also accepts `attemptsTable`, `eventsTable`, and `recoveriesTable`; subscription storage accepts `exclusionsTable`. If you customize table names, update your migrations and related table settings to match. Public APIs use `notifiableType`, `notifiableId`, and `channelName`; existing database column names stay the same.

### Sending from a worker

Your worker passes the saved delivery ID to `NotificationDeliveryService`:

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
