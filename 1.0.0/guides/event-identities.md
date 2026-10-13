# Event Identities

Imagine your application queues a notification when an order is ready. If the request is repeated, you probably want the original notification back instead of sending another one. An event key gives Megaphone a way to recognize that repeat.

### Choosing an event key

Pass your event identity in the `options` struct when calling `plan()` or `queue()`:

```js
var publication = megaphone.queue(
    notifiables = user,
    notification = "OrderReadyNotification",
    properties = { "orderId": orderId },
    options = {
        "namespace": "Orders",
        "eventKey": orderId,
        "version": 1,
        "occurredAt": orderReadyDate
    }
);
```

The `namespace`, `eventKey`, and `version` together identify the event. Repeating that identity with the same payload hash returns the original work. It doesn't add new recipients, channels, or devices. If the saved payload differs, Megaphone rejects the replay.

You can leave these options out when you want a new event for every call:

| Option | Default | Purpose |
| --- | --- | --- |
| `namespace` | `megaphone` | Groups event keys belonging to the same application feature. |
| `eventKey` | Generated UUID | Identifies this occurrence within the namespace. |
| `version` | `1` | Distinguishes versions of the event. Must be a positive integer. |
| `occurredAt` | Now | Records when the event happened. |
| `availableDate` | Now | Sets when deliveries become available to a worker. |
| `admitAfter` | Not set | Excludes occurrences before this cutoff. |

If `occurredAt` is earlier than `admitAfter`, queueing returns an expired publication without saving work. An occurrence exactly at the cutoff is allowed.

### Saving a plan

Sometimes your application needs to save its decision to send before a worker can queue the deliveries. Use `plan()` to capture that decision and save its memento with your application record:

```js
var plan = megaphone.plan(
    notifiables = user,
    notification = "OrderReadyNotification",
    properties = { "orderId": orderId },
    options = { "namespace": "Orders", "eventKey": orderId }
);
var savedPlan = serializeJSON( plan.getMemento() );
```

Add a method provider to the service that restores your saved plans:

```js
/** Supplies a fresh plan through WireBox. */
private megaphone.models.NotificationPlan function newNotificationPlan()
    provider="NotificationPlan@megaphone" {
}
```

Later, restore the plan and queue it:

```js
var plan = newNotificationPlan()
    .populateFromMemento( deserializeJSON( savedPlan ) );
var publication = megaphone.queuePlan( plan );
```

WireBox supplies an empty plan; `populateFromMemento()` fills it with the saved values. The memento has `schemaVersion=1`, an `event`, a `deliveries` array, and an optional `admitAfter`. It keeps the original Notifiable identities, channel choices, device IDs, payload, occurrence time, availability dates, and hashes. Restoring it doesn't ask `via()` or preferences to choose a new audience.

For example, a user who registers another device after the plan was saved won't receive the old notification on that device. A channel that was suppressed in the plan stays suppressed.

{% hint style="info" %}
`plan()` also accepts optional `connection` and `queue` arguments. Its version 1 memento stores explicit choices in `queueOptions`; existing saved plans without that field use the current defaults. You can override either choice when calling `queuePlan()` without changing the captured audience or hashes.

A trusted producer that already saved and validated its original admission can call `queuePlan(plan=plan, alreadyAdmitted=true)`. This bypasses the admission cutoff when handing off that saved work. It keeps the plan's captured recipients and channels.
{% endhint %}

### Keeping hashes stable

Megaphone's default snapshot hasher sorts object keys and preserves array order. If your application already has persisted event hashes, keep using the same algorithm when upgrading.

You can configure a `snapshotHasher` mapping under `queue`. Its `digest(value, context)` method receives a context containing `namespace` and `kind`, where `kind` is `payload` or `routing`. This lets your application retain its existing hash rules for replay comparisons.

See [BaseNotification](../reference/basenotification.md#getqueueproperties) for choosing the properties saved in a plan.
