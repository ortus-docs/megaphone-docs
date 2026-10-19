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
| `connection` | `string` | false | | A registered cbq connection. Overrides `queue.defaultConnection`; otherwise cbq uses its default connection. |
| `queue` | `string` | false | | A queue on the selected connection. Overrides `queue.defaultQueue`; otherwise cbq uses the selected connection's default queue. |

With cbq's standard defaults, omitted `connection` and `queue` arguments both use `default`. Configured `queue.defaultConnection` and `queue.defaultQueue` settings or custom cbq defaults take precedence.

**Return**: A `NotificationPlan`. See [Event Identities](../guides/event-identities.md) for the available options and saving its memento.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.UnsupportedDurableProvider` if a selected channel supports neither durable transport nor the collection protocol.
{% endhint %}

### queue

Plans and saves a Notification for delivery by a worker. It accepts the same arguments as `plan()`.

Install and configure cbq in your application to use the bundled adapter. If cbq is unavailable when it tries to enqueue, Megaphone throws `Megaphone.Queue.MissingCBQ`.

The bundled cbq adapter dispatches a delivery job to the configured queue, using the Provider on its registered connection. Megaphone saves the event and deliveries transactionally; an enqueue failure rolls back those records. Queue-side rollback and execution timing follow the chosen Provider. A database queue on the same datasource can join the transaction. Repeating an event identity with the same payload hash returns its original work.

**Return**: A `NotificationPublication`, with `getEvent()`, `getDeliveries()`, and `getCollections()` accessors. Collection channels persist admissions instead of transport deliveries; `getCollections()` returns `{id, status, created}` records separately. See [Durable Delivery](../guides/durable-delivery.md) for cbq setup and examples. Its final section covers [custom queue adapters](../guides/durable-delivery.md#making-a-custom-queue-adapter).

### queuePlan

Queues a previously captured `NotificationPlan`. Use this when your application saved a plan before handing it to a worker.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `plan` | `NotificationPlan` | true | | The original plan, including its captured recipients and channel choices. |
| `alreadyAdmitted` | `boolean` | false | `false` | Bypasses the admission cutoff for a trusted producer that already saved and validated admission. |
| `connection` | `string` | false | | Overrides the plan's saved cbq connection for this handoff. |
| `queue` | `string` | false | | Overrides the plan's saved queue for this handoff. |

The plan retains explicit `connection` and `queue` arguments in its optional `queueOptions` memento. cbq supplies omitted route and worker settings.

Queueing a saved plan keeps its original audience, payload, times, and hashes. It doesn't enable a channel that was suppressed in the plan.

**Return**: A `NotificationPublication`. An occurrence before the plan's admission cutoff returns an expired publication without saving work, unless `alreadyAdmitted` is true.

### planMany

`planMany(requests)` accepts an ordered array of request structs. Each request uses the same named arguments as `plan()`, including a Notification instance or WireBox mapping, its Notifiables, properties, options, and queue routing.

**Return:** An array of `NotificationPlan` objects in request order. This operation does not write or send. Use a distinct Notification instance per request when preloading transient application preferences.

### queuePlans

`queuePlans(plans, alreadyAdmitted=false, connection?, queue?)` saves and enqueues an ordered array of plans in one datasource transaction. A failure in any batch rolls back the whole operation. The queue adapter must participate in the same transaction for queue atomicity.

**Return:** Ordered `NotificationPublication` objects. Existing event identities return their original audience and delivery states without re-enqueueing work. Admission cutoffs, queue overrides, and trusted `alreadyAdmitted` behavior match `queuePlan()`.

### admitPlans and admitRequests

`admitPlans(plans, alreadyAdmitted=false, connection?, queue?)` saves and enqueues plans without returning publication objects. `admitRequests(requests, alreadyAdmitted=false, connection?, queue?)` accepts the same request structs as `planMany()` without returning plans or publications.

**Return:** Void. The optimized path requires a queue adapter implementing `enqueueReferencesMany(references, connection?, queue?)`; each reference contains `id`, `state`, and `availableDate`. Unsupported adapters, collection plans, and applications with named digests configured use the existing mapped-model path before writing. Identity conflicts, cutoffs, and transaction rollback still apply.

These batch methods were added in 1.3.0. See [Queueing Several Notifications](../guides/durable-delivery.md#queueing-several-notifications) for an example and optional queue adapter methods.

### Collection channels

An [EmailDigestProvider](../providers/emaildigestprovider.md) channel uses the same `notify()`, `plan()`, `queue()`, and `queuePlan()` entry points. Planning captures the item; publication collects it transactionally. Scheduled collection does not enqueue a mail job; a due digest later creates its own durable email delivery. Immediate collection freezes one item and creates a delivery for publication to enqueue. It uses the same digest renderer and needs no separate ordinary-email route.

A saved plan retains its collection capability. Changing that channel to a transport Provider before publication throws `Megaphone.Digests.CollectionChannelChanged`, so saved collection content cannot silently switch to an unrelated transport Provider. Event replay returns its saved collection admissions, even when a later plan offers fewer channels.

In 1.3.0, [DigestProvider](../providers/digestprovider.md) uses the same collection protocol with `toDigest()` and a configured `properties.digestName`. Its summary is an ordinary Notification that chooses its own delivery channels. A generated summary cannot route into another collection channel. See [Digests](../guides/digests.md).
