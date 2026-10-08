# BaseProvider

### getProviderName

Returns the name for this Provider.

{% hint style="warning" %}
This name should be the same across all different channels using the same Provider.
{% endhint %}

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (string) The Provider name

### notify

Sends a [`Notification`](../basenotification.md) to a [`Notifiable`](../inotifiable.md) through this Channel instance using the configured Provider.

This method will be called once for each [`Notifiable`](../inotifiable.md) receiving the notification, even if the [`NotificationService#notify`](../notificationservice.md#notify) method was called with multiple [`Notifiable`](../inotifiable.md) instances.

{% hint style="warning" %}
When implementing this method, make sure not to modify the [`Notification`](../basenotification.md) instance as this instance will be used for each [`Notifiable`](../inotifiable.md).
{% endhint %}

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th width="123">Required</th><th width="58">Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../inotifiable.md"><code>Notifiable</code></a> instance receiving the <a href="../basenotification.md"><code>Notification</code></a>.</td></tr><tr><td>notification</td><td><a href="../basenotification.md"><code>BaseNotification</code></a></td><td>true</td><td></td><td>The Notification instance to send to the notifiable.</td></tr></tbody></table>

**Return**: [`Notification`](../basenotification.md) instance, after it has been sent to the [`INotifiable`](../inotifiable.md).

### routeNotificationFor

Determines the route the notification should be sent to by calling the `routeNotificationFor{ChannelType}` method, if it exists.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>type</td><td><code>string</code></td><td>true</td><td></td><td>The type of channel to route to.</td></tr><tr><td>notifiable</td><td><a href="../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The notifiable instance the notification is being sent to.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>true</td><td></td><td>The name of the channel to route to.</td></tr><tr><td>additionalArgs</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>Any additional arguments to pass to the routing method (<code>to{ChannelType}</code> method).</td></tr><tr><td>optional</td><td><code>boolean</code></td><td>false</td><td><code>false</code></td><td>Boolean flag determining if the routing method is optional or not.</td></tr></tbody></table>

**Return**: The result of the `routeNotificationFor{ChannelType}` method. The variable type depends on the Provider.
