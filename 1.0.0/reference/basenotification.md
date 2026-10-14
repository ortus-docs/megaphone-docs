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

### getQueueProperties

Returns the properties to save when this Notification is planned for durable delivery. By default, these are the properties explicitly passed to `populate()`, including those supplied to `plan()` or `queue()`.

If you configure the Notification through setters, override this method to include those values:

```js
public struct function getQueueProperties() {
    return {
        "stockSymbol": getStockSymbol(),
        "completionTimestamp": getCompletionTimestamp()
    };
}
```

Use JSON-safe data that a worker can use to reconstruct the Notification. Resolve users and build mail or request objects when preparing delivery, instead of saving those objects or credentials in the snapshot.

**Return**: A `struct` of Notification properties.

### getChannelPreferences

Returns the preferences for the channels selected by `via()`. Override it when your application needs to resolve channel choices for this Notifiable.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The Notifiable receiving the Notification. |
| `channelNames` | `array` | false | Result of `via(notifiable)` | The channels whose preferences should be returned. |

Each channel has an `enabled` value and a `source` describing where the choice came from:

```js
return {
    "database": { "enabled": true, "source": "default" },
    "email": { "enabled": false, "source": "team:42" }
};
```

The default enables every selected channel. `notify()` skips disabled channels; durable planning captures them as suppressed work. See [Notification Preferences](../guides/notification-preferences.md) for resolving application scopes.

**Return**: A `struct` keyed by channel name, with `{ enabled, source }` for each channel.

### getNotificationMapping

Returns the WireBox mapping used to reconstruct this Notification in a worker. It defaults to the WireBox identity, or the component name when no WireBox identity is available.

Override this if your stored notification type differs from the mapping needed to create the Notification.

**Return**: A `string` reconstruction mapping.

### getDeliveryContext

Returns application routing data for a Notifiable and channel, separately from the Notification's queued properties. Override it when the delivery needs context such as a specific push device.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The Notifiable receiving the Notification. |
| `channelName` | `string` | true | | The configured channel name. |

**Return**: A JSON-safe `struct`. The default is `{}`.
