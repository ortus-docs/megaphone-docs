## Durable notifications

Use the same notifiable and notification conventions for immediate and queued delivery:

```js
megaphone.notify(user, "OrderReadyNotification", { "orderId": orderId });
var publication = megaphone.queue(
    notifiables = user,
    notification = "OrderReadyNotification",
    properties = { "orderId": orderId },
    options = { "namespace": "Orders", "eventKey": orderId, "version": 1 }
);
publication.getEvent().getId();
publication.getDeliveries()[1].getId();
```

`NotificationService.plan()` accepts the same arguments and returns a `NotificationPlan` without writing or sending. `queue()` plans and stores work in the caller's datasource transaction. `SendsNotifications` also provides `queue()`.

Options are `namespace` (default `megaphone`), `eventKey` (generated if omitted), `version` (1), `occurredAt` and `availableDate` (now), and optional `admitAfter`. Occurrences older than the cutoff return an expired publication without recording work. Equality is admitted.

Persist `plan.getMemento()` when admission must precede worker handoff. Restore it with `new megaphone.models.NotificationPlan({}, []).populateFromMemento(saved)` and call `queuePlan(plan)`. Its version 1 memento contains `event`, `deliveries`, and optional `admitAfter`. It preserves identities, payloads, hashes, destinations, and channel choices. A trusted producer may use `queuePlan(plan, alreadyAdmitted=true)` after validating its original persisted admission. This bypasses the replay cutoff; it does not replan the audience.

Notifications extend `BaseNotification`, implement `via(notifiable)` and the usual `to{Provider}` methods, and may override `getChannelPreferences(notifiable)` to return a channel-name map of `{ enabled, source }`. The default enables the channels returned by `via()`. Synchronous delivery skips disabled channels. Durable delivery records them as suppressed; later opt-in does not revive them.

`getQueueProperties()` defaults to the explicitly populated properties. Notifications configured through setters must override it to return their durable properties. Store JSON data rather than components, closures, credentials, users, or prepared mail/request objects. `getNotificationMapping()` defaults to the WireBox identity; override it when notification type and reconstruction mapping differ. `getDeliveryContext(notifiable, channelName)` provides application routing context, separate from the notification properties.

### Module configuration

```js
moduleSettings.megaphone = {
    "channels": {
        "database": { "provider": "DatabaseProvider@megaphone", "properties": {} },
        "email": { "provider": "EmailProvider@megaphone", "properties": { "durableAdapter": "ActivityMailAdapter" } },
        "push": { "provider": "WebPushProvider@megaphone", "properties": { "runtime": "ApplicationPushRuntime" } }
    },
    "storage": {
        "events": {}, "deliveries": {}, "preferences": {}, "subscriptions": {}
    },
    "durable": {
        "queueAdapter": "NotificationQueue",
        "notifiableResolver": "NotifiableResolver",
        "eligibilityPolicy": "NotificationEligibility",
        "snapshotHasher": "NotificationSnapshotHasher@megaphone",
        "leaseSeconds": 60, "maxAttempts": 5, "retrySeconds": 30,
        "maxRecoveryAttempts": 5, "maxRecoveryAllowance": 45
    }
};
```

Storage settings accept `table`, `queryOptions`, and `datasource`. Deliveries also accept `attemptsTable`, `eventsTable`, and `recoveriesTable`; subscriptions accept `exclusionsTable`. Adapt migrations when using custom table names, and keep related table settings consistent. New public APIs use `notifiableType`, `notifiableId`, and `channelName`; existing storage columns remain unchanged.

A queue adapter implements `enqueue(delivery)` and stores only `delivery.getId()` in the same datasource transaction. It must not send there. Queue insertion failure rolls back publication. Without an adapter, schedule `NotificationDeliveryService.getDueDeliveries()` and `recoverExpired()`.

A notifiable resolver implements `resolve(type, id)` and returns the matching current notifiable or null. An optional durable.deliveryPolicy mapping implements getDispatchOptions(id) for configured lease, attempt, and retry options. Explicit dispatch options take precedence.

An optional eligibility policy implements `check(delivery, notifiable, event)` and returns a `DeliveryPreparation` with status `ready`, `suppressed`, `deferred`, or `permanent`, plus optional reason and retry date. Application authorization, source revision, and feature checks belong here. Preferences are checked before transport too.

The default snapshot hasher sorts object keys and preserves array order. An application hasher implements `digest(value, context)`, where context includes `namespace` and `kind` (`payload` or `routing`). Retain an existing canonical hash algorithm when upgrading persisted events.

### Providers and dispatch

Providers may implement the optional `IDurableNotificationProvider` capability:

- `prepareDelivery(notifiable, notification, delivery)` returns a `DeliveryPreparation`; prepare content without provider I/O.
- `sendDelivery(prepared, delivery)` returns a `DeliveryResult` with status `accepted`, `retryable`, `permanent`, or `ambiguous` and optional reason, provider reference, and retry date.

Use `prepared.getData()` for transient transport objects. Its memento contains status metadata, not transport data. Existing synchronous `notify()` still returns a `BaseNotification`. A provider lacking durable support is rejected during planning. Durable email requires a configured adapter that supplies explicit acceptance evidence.

`NotificationDeliveryService.dispatch(id, options={})` resolves the saved notification and registered channel, checks current eligibility, prepares delivery, checks again, and records transport start before sending. It returns a `DeliveryResult`. Options can override lease, attempt, and retry settings.

Duplicate jobs cannot share a lease. Preparation failures and feature pauses do not consume transport allowance. An exception after transport starts is ambiguous and cannot trigger an automatic resend. Acceptance does not establish receipt or read state. Do not wrap provider I/O in an application transaction.
