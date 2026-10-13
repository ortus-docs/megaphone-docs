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

### plan

Captures the recipients, channels, preferences, devices, and properties for a queued Notification without saving or sending it. Like `notify()`, it accepts a Notification instance or a WireBox name.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiables` | `INotifiable` or array | true | | The Notifiables that should receive the Notification. |
| `notification` | `string` or `BaseNotification` | true | | The Notification instance or its WireBox mapping. |
| `properties` | `struct` | false | `{}` | Properties to populate on the Notification. |
| `options` | `struct` | false | `{}` | Event identity, occurrence time, availability date, and optional admission cutoff. |

**Return**: A `NotificationPlan`. See [Event Identities](../guides/event-identities.md) for the available options and saving its memento.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.UnsupportedDurableProvider` if a selected channel does not support durable delivery.
{% endhint %}

### queue

Plans and saves a Notification for delivery by a worker. It accepts the same arguments as `plan()`.

The bundled cbq adapter dispatches a delivery job to the configured queue, using the Provider on its registered connection. Megaphone saves the event and deliveries transactionally; an enqueue failure rolls back those records. Queue-side rollback and execution timing follow the chosen Provider. A database queue on the same datasource can join the transaction. Repeating an event identity with the same payload hash returns its original work.

**Return**: A `NotificationPublication`, with `getEvent()` and `getDeliveries()` accessors. See [Durable Delivery](../guides/durable-delivery.md) for cbq setup and examples. Its final section covers [custom queue adapters](../guides/durable-delivery.md#making-a-custom-queue-adapter).

### queuePlan

Queues a previously captured `NotificationPlan`. Use this when your application saved a plan before handing it to a worker.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `plan` | `NotificationPlan` | true | | The original plan, including its captured recipients and channel choices. |
| `alreadyAdmitted` | `boolean` | false | `false` | Bypasses the admission cutoff for a trusted producer that already saved and validated admission. |

Queueing a saved plan keeps its original audience, payload, times, and hashes. It doesn't enable a channel that was suppressed in the plan.

**Return**: A `NotificationPublication`. An occurrence before the plan's admission cutoff returns an expired publication without saving work, unless `alreadyAdmitted` is true.
