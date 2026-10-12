# BaseNotification

### via

Returns an array of channel names to send this notification on.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The notifiable instance being sent this notification instance.</td></tr></tbody></table>

**Return**: An array of `string` channel names.

### populate

Populates the passed in struct of properties onto the variables scope of the Notification.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>properties</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>The properties to populate.</td></tr></tbody></table>

**Return**: The Notification instance.

### routeForType

Routes the notification to the correct `to{ChannelType}` method.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>type</td><td><code>string</code></td><td>true</td><td></td><td>The type of channel to route to.</td></tr><tr><td>notifiable</td><td><a href="inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The notifiable instance the notification is being sent to.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>true</td><td></td><td>The name of the channel to route to.</td></tr><tr><td>additionalArgs</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>Any additional arguments to pass to the routing method (<code>to{ChannelType}</code> method).</td></tr></tbody></table>

**Return**: The result of the routing method. The variable type depends on the Provider.

### getNotificationType

Returns the WireBox ID of the Notification.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (string) The WireBox ID of the Notification.

Notifications extend `BaseNotification`, implement `via(notifiable)` and the usual `to{Provider}` methods, and may override `getChannelPreferences(notifiable)` to return a channel-name map of `{ enabled, source }`. The default enables the channels returned by `via()`. Synchronous delivery skips disabled channels. Durable delivery records them as suppressed; later opt-in does not revive them.

`getQueueProperties()` defaults to the explicitly populated properties. Notifications configured through setters must override it to return their durable properties. Store JSON data rather than components, closures, credentials, users, or prepared mail/request objects. `getNotificationMapping()` defaults to the WireBox identity; override it when notification type and reconstruction mapping differ. `getDeliveryContext(notifiable, channelName)` provides application routing context, separate from the notification properties.
