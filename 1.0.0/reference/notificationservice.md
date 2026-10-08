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
