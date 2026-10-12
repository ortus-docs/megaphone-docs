# NotificationService

### notify

Sends a notification to a single notifiable instance or an array of notifiable instances.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiables</td><td><a href="inotifiable.md"><code>INotifiable</code></a> | <code>array&#x3C;</code><a href="inotifiable.md"><code>INotifiable</code></a><code>></code></td><td>true</td><td></td><td>A single Notifiable instance or an array of Notifiable instances that will receive the notification.</td></tr><tr><td>notification</td><td><code>string</code> | <a href="basenotification.md"><code>BaseNotification</code></a></td><td>true</td><td></td><td>The notification instance or a string WireBox mapping to send to the notifiables.</td></tr><tr><td>properties</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>A struct of properties to populate the notification with.</td></tr></tbody></table>

**Return**: `notification` instance, after it has been populated and sent to the `notifiables`.

### getChannel

Returns a channel instance by name.

<table><thead><tr><th width="149">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channelName</td><td><code>string</code></td><td>true</td><td></td><td>The name of the channel to return.</td></tr></tbody></table>

**Return**: A [`BaseProvider`](baseprovider/) instance for the channel name.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChanel`
{% endhint %}

### getChannels

Returns a map of channel names to Provider instances.

<table><thead><tr><th width="149">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: A `struct` map of channel names to Provider instances.

## Durable notifications

Use the same notifiable and notification conventions for immediate and queued delivery:

```cfc
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
