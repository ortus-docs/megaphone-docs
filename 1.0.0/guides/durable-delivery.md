# Durable Delivery

Want to save a Notification now and let a worker send it later? Megaphone includes a cbq adapter and a delivery job for that. Install cbq in your application when you want to use them. You use the same Notifications, Notifiables, and channels as `notify()`, then call `queue()` to hand the work to cbq.

### Setting up cbq

cbq is an optional application dependency. Megaphone does not install it for you:

```js
box install cbq
```

If you try to enqueue through Megaphone's cbq adapter without an available, active cbq module, it throws `Megaphone.Queue.MissingCBQ`. You can use synchronous notifications, a custom queue adapter, or a polling worker without installing cbq.

Megaphone uses cbq's default connection and that connection's default queue. To use another registered connection or queue by default, give Megaphone their names. cbq supplies the Provider and worker settings. You'll also need to tell Megaphone how to load a Notifiable in the worker:

```js
moduleSettings.megaphone = {
    "channels": {
        "database": {
            "provider": "DatabaseProvider@megaphone",
            "properties": {}
        }
    },
    "queue": {
        "defaultConnection": "messages",
        "defaultQueue": "notifications",
        "notifiableResolver": "NotifiableResolver"
    }
};
```

`NotifiableResolver` is your application component. Its `resolve(type, id)` method returns the current Notifiable, or null if it no longer exists. Megaphone uses it to load the recipient when the job runs.

Register the connection and workers in your application's cbq configuration. Megaphone uses those registrations and leaves the Provider choice to cbq. If you want a different connection from cbq's default, set `queue.defaultConnection` to its registered name. Set `queue.defaultQueue` to choose the queue used for notification jobs.

Apply Megaphone's migrations through your application's migration ledger before queueing. Follow your chosen queue Provider's setup instructions too. For cbq's database Provider, that includes its migrations from `resources/database/migrations`; other Providers have their own requirements.

For a cbq Provider that uses worker pools, add a worker for the notification queue. This example uses a connection named `messages` with a default queue named `notifications`; use your registered connection and queue names:

```js
newWorkerPool( "notifications" )
    .forConnection( "messages" )
    .onQueue( "notifications" )
    .quantity( 1 )
    .timeout( 60 )
    .maxAttempts( 1 );
```

Each job carries only the delivery ID. Megaphone loads the saved Notification and decides whether another attempt is allowed after a known retryable, deferred, or busy outcome. It releases the job through cbq with the appropriate backoff, so the registered Provider handles timing. Unhandled errors do not get an automatic extra attempt, and an ambiguous outcome never schedules another send.

The bundled adapter works with the Provider on your registered connection. That includes synchronous execution when you choose cbq's synchronous Provider. For background Providers, use cbq's `registerWorkers=false` setting on a process that only publishes jobs.

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

Want to choose a connection or queue for one Notification? Pass `connection` and `queue`, just as you would with cbq:

```js
var publication = megaphone.queue(
    notifiables = user,
    notification = "StockRebalancingCompleteNotification",
    properties = { "stockSymbol": "APPL", "completionTimestamp": now() },
    connection = "messages",
    queue = "stock-updates"
);
```

You can pass either argument on its own. A `connection` argument takes precedence over `queue.defaultConnection`; omitting both uses cbq's default connection. A `queue` argument takes precedence over `queue.defaultQueue`. When both are omitted, cbq uses the selected connection's default queue. Configure worker timeouts through cbq.

With no Megaphone route overrides and cbq's standard defaults, omitted `connection` and `queue` arguments both use `default`. If you configure another default connection or a different default queue on that connection, Megaphone follows those cbq settings.

`plan()` accepts these arguments too and saves them with the plan. `queuePlan()` uses the saved choices unless you pass a new `connection` or `queue` at handoff. Replaying an event returns its original work without adding jobs to a different queue.

Megaphone saves the event and deliveries in a transaction and asks cbq to enqueue each delivery ID. An enqueue failure rolls back those Megaphone records.

When cbq's database Provider uses the same datasource, job insertion joins that transaction too. External queue Providers follow their own commit behavior: a database rollback can't remove an already published external job or undo a synchronous send. For external workers, arrange the commit handoff so they can see the saved delivery before consuming its ID.

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

Use `queue.defaultConnection` and `queue.defaultQueue` to choose the default route for notification jobs. Keep Providers and worker timeouts in your application's cbq configuration; use `connection` and `queue` arguments when an individual call needs a different route.

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
    "queue": {
        "defaultConnection": "",
        "defaultQueue": "",
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

Leave `defaultConnection` and `defaultQueue` empty to use cbq's default connection and that connection's default queue. The resolver and eligibility mappings are your application components; `NotificationSnapshotHasher` is bundled. Here is what the application callbacks do:

| Setting | Method | What your application supplies |
| --- | --- | --- |
| `notifiableResolver` | `resolve(type, id)` | The current Notifiable, or null if it no longer exists. |
| `eligibilityPolicy` | `check(delivery, notifiable, event)` | A `DeliveryPreparation` describing whether this notification can still be sent. |
| `deliveryPolicy` | `getDispatchOptions(id)` | Optional lease, attempt, and retry settings for a delivery. |
| `snapshotHasher` | `digest(value, context)` | A stable hash when your application needs its own hashing rules. |

### Loading a Notifiable

Megaphone saves the Notifiable's type and ID with the delivery. Your resolver uses them to load the current application object when the worker runs. For an application whose Users return `"User"` from `getNotifiableType()`, the resolver could look like this:

```js
// models/NotifiableResolver.cfc
component singleton {

    property name="users" inject="UserService";

    public any function resolve( required string type, required string id ) {
        if ( arguments.type != "User" ) {
            return;
        }
        return variables.users.find( arguments.id );
    }

}
```

`UserService` belongs to your application. In this example, its `find(id)` method returns a User or null when the User no longer exists. Use your own lookup method, and add a branch for each Notifiable type your application supports. Return null for a missing recipient; Megaphone suppresses that delivery without sending it.

The returned object must implement `getNotifiableType()` and `getNotifiableId()`, along with any routing methods your Notification uses. Megaphone checks that its type and ID match the saved delivery.

### Application policies

Use the eligibility policy for application rules that may have changed since planning, such as access to the underlying record or whether a feature is enabled. Return a `DeliveryPreparation` with a status of `ready`, `suppressed`, `deferred`, or `permanent`, and an optional reason and retry date. Megaphone also checks current channel preferences before sending.

For example, you might skip notifications for inactive accounts and pause delivery while a feature is disabled:

```js
// models/NotificationEligibility.cfc
component singleton {

    property name="featureFlags" inject="FeatureFlags";

    /** Checks the account and feature policy before starting delivery. */
    public megaphone.models.Delivery.DeliveryPreparation function check(
        required any delivery,
        required any notifiable,
        required any event
    ) {
        if ( !arguments.notifiable.getIsActive() ) {
            return newDeliveryPreparation().configure( { "status": "suppressed", "reason": "account-inactive" } );
        }
        if ( !variables.featureFlags.isEnabled( "notifications" ) ) {
            return newDeliveryPreparation().configure( { "status": "deferred", "reason": "notifications-paused", "retryDate": dateAdd( "n", 5, now() ) } );
        }
        return newDeliveryPreparation().configure( { "status": "ready" } );
    }

    /** Supplies a fresh preparation decision through WireBox. */
    private megaphone.models.Delivery.DeliveryPreparation function newDeliveryPreparation()
        provider="DeliveryPreparation@megaphone" {
    }

}
```

`FeatureFlags.isEnabled(name)` and the User's `getIsActive()` method are application methods in this example. Replace them with your own checks. You can also use `event.getPayload()` and `delivery.getRoutingData()` to check access to the record behind the Notification.

Register these components through `queue.notifiableResolver = "NotifiableResolver"` and `queue.eligibilityPolicy = "NotificationEligibility"`, as shown above. The eligibility policy is optional. Megaphone checks it before preparing content and again before starting transport, so keep it safe to call more than once. Return `deferred` for a temporary pause; that does not consume a transport attempt.

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

An accepted delivery means the notification Provider accepted it. It doesn't mean the user has received or read it. Background queue Providers let the worker send after publication; a synchronous queue Provider executes the job inline when it is enqueued.

### Supporting durable delivery in a Provider

Providers can support the optional `IDurableNotificationProvider` protocol with two methods:

| Method | Return | Purpose |
| --- | --- | --- |
| `prepareDelivery(notifiable, notification, delivery)` | `DeliveryPreparation` | Build and validate the content before making a transport request. |
| `sendDelivery(prepared, delivery)` | `DeliveryResult` | Send the prepared content and report the outcome. |

`DeliveryResult` supports `accepted`, `retryable`, `permanent`, and `ambiguous`, with optional reason, Provider reference, and retry date. Use `prepared.getData()` for temporary mail or request objects. The preparation memento contains status metadata, so those transport objects aren't serialized.

The Database and Web Push Providers support durable delivery. For email, configure the Email Provider's `durableAdapter` with a component that returns a `DeliveryResult` based on the mail transport's response.

For example, a provider message ID returned after accepting a message can be saved as `providerReference`. A timeout after submission is different: the provider may have accepted the message even though the application didn't receive its response. Return `ambiguous` in that case so Megaphone won't send a duplicate automatically.

cbmailservices exposes send results through its [Mail callbacks](https://coldbox-mailservices.ortusbooks.com/essentials/sending-mail). The meaning of success depends on the mail protocol. It can mean that a provider accepted the message, that the engine queued it locally, or that a test protocol captured it. A method returning successfully by itself doesn't distinguish those outcomes. The current Megaphone adapter hook lets applications supply transport-specific handling or use an existing receipt ledger.

You can keep using a Provider that only implements `notify()` for synchronous notifications. If you try to plan a durable notification for that channel, Megaphone throws `Megaphone.Configuration.UnsupportedDurableProvider`.

### Making a custom queue adapter

If your application needs a different job queue or extra work when inserting a job, you can supply your own adapter. Implement `INotificationQueue` with an `enqueue(delivery, connection, queue)` method that receives a `NotificationDelivery` object and optional route arguments. Megaphone forwards `connection` and `queue` only when a caller supplies them.

Here is an example using an application queue service:

```js
// models/NotificationQueueAdapter.cfc
component singleton implements="megaphone.models.Interfaces.INotificationQueue" {

    property name="jobQueue" inject="JobQueue";

    /** Queues a saved delivery reference for the application worker. */
    public void function enqueue( required megaphone.models.Delivery.NotificationDelivery delivery, string connection, string queue ) {
        var jobArguments = {
            "job": "DeliverNotification",
            "properties": { "deliveryId": arguments.delivery.getId() },
            "availableDate": arguments.delivery.getAvailableDate()
        };
        for ( var name in [ "connection", "queue" ] ) {
            if ( !isNull( arguments[ name ] ) ) {
                jobArguments[ name ] = arguments[ name ];
            }
        }
        variables.jobQueue.enqueue( argumentCollection = jobArguments );
    }

}
```

`JobQueue` and its `enqueue()` method in this example belong to your application. Let enqueue failures propagate so Megaphone can roll back the event and deliveries. A database queue can join the publication transaction; an external queue needs its own commit handoff, just as it does when used through cbq. The adapter queues the ID; sending happens later in the worker.

Register the adapter in your module settings:

```js
moduleSettings.megaphone.queue.queueAdapter = "NotificationQueueAdapter";
```

You can also supply an adapter instance. Without an override, Megaphone uses `CBQNotificationQueue@megaphone`.

Your worker calls `NotificationDeliveryService.dispatch(deliveryId)` and uses the returned `DeliveryResult`. If you schedule another job after a `retryable`, `deferred`, or `busy` result, reload the delivery and respect its current state and availability date. For a busy delivery, wait until its lease expires. Let Megaphone control the transport allowance, and never automatically resend an accepted or ambiguous delivery. See [Delivery Operations](delivery-operations.md) for recovery and receipt reconciliation.

If you prefer a scheduled polling worker, explicitly set `queue.queueAdapter` to an empty string. Megaphone still saves deliveries; your worker can find them with `getDueDeliveries()` and handle expired leases with `recoverExpired()`.

### Collecting a digest

A [DigestProvider](../providers/digestprovider.md) channel first admits the Notification into a named collection using `toDigest()`. Scheduled subscriptions wait for a cutoff; immediate subscriptions create a one-item summary. `plan()` captures the item, and `queuePlan()` persists that collection in the publication transaction. The publication's `getCollections()` admissions are separate from `getDeliveries()`; collection never means Provider acceptance.

At the cutoff, Megaphone publishes the configured summary Notification through normal durable routing. Its `via()` selects delivery channels, and each channel gets an independent delivery record and retry history. Repeating cutoff dispatch cannot publish a second summary for the batch. The summary cannot route back into a collection channel.

Use [Digests](digests.md) to configure subscriptions, consent and eligibility, selection, summary Notifications, and the native ColdBox scheduler. Summary deliveries use the same claims, attempts, and receipt reconciliation described here.

{% hint style="warning" %}
[EmailDigestProvider](../providers/emaildigestprovider.md) is deprecated in Megaphone 1.3.0 in favor of [DigestProvider](../providers/digestprovider.md). Existing email-channel and renderer definitions remain supported. See [Named Email Digests](email-digests.md) for that legacy path.
{% endhint %}

### Queueing several Notifications

Megaphone 1.3.0 adds `planMany()` and `queuePlans()` for publishing several Notifications in one transaction. Each request uses the same arguments as `plan()`:

```js
var plans = notificationService.planMany( [
    {
        "notifiables": firstUser,
        "notification": "ReceiptNotification",
        "properties": { "receiptId": "one" },
        "options": { "eventKey": "receipt-one" }
    },
    {
        "notifiables": secondUser,
        "notification": "ReceiptNotification",
        "properties": { "receiptId": "two" },
        "options": { "eventKey": "receipt-two" }
    }
] );

var publications = notificationService.queuePlans(
    plans = plans,
    connection = "database",
    queue = "notifications"
);
```

`planMany()` returns one `NotificationPlan` per request without writing or sending. `queuePlans()` returns the corresponding `NotificationPublication` objects in the same order. A failure rolls back the whole batch and its caller's transaction. Queue insertion must use the same transactional datasource when you need queue writes to roll back with publication.

Replaying an event keeps its original audience, suppression choices, and delivery state. It does not add recipients or enqueue the same work again. Plans involving collections, and applications with named digests configured, keep the existing publication path within the shared transaction so admission history and replay rules stay intact.

If you do not need publication results, use `admitPlans(plans, ...)`. If you do not need saved plans either, `admitRequests(requests, ...)` accepts the request array directly. Both return void. Their optimized path requires a queue adapter with `enqueueReferencesMany()`; existing adapters and digest plans retain the normal mapped-model path. See the [NotificationService reference](../reference/notificationservice.md#planmany) for the arguments.

Custom queue adapters can implement `enqueueMany(deliveries, connection?, queue?)` to receive a batch of public `NotificationDelivery` objects. Adapters that only implement `enqueue()` continue to work. The bundled cbq adapter uses bulk dispatch for immediately available jobs and keeps delayed scheduling for future jobs. Batching depends on the configured cbq Provider and version.

`enqueueReferencesMany(references, connection?, queue?)` is a separate opt-in for the void admission methods. Each reference contains `id`, `state`, and `availableDate`. The adapter must preserve its normal queue interception and delayed scheduling behavior.
