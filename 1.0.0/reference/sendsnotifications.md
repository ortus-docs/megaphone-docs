# SendsNotifications

This is a [delegate component](https://wirebox.ortusbooks.com/usage/wirebox-delegators) and requires ColdBox 7+. It should be added to an [`INotifiable`](inotifiable.md) instance.

```js
component
    name="User"
    delegates="SendsNotifications@megaphone"
    accessors="true"
{

    property name="id";

    public string function getNotifiableId() {
        return getId();
    }

    public string function getNotifiableType() {
        return "User";
    }

}
```

### notify

Sends a notification to this [`INotifiable`](inotifiable.md) instance.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notification</td><td><code>string</code> | <a href="basenotification.md"><code>BaseNotification</code></a></td><td>true</td><td></td><td>The notification instance or a string WireBox mapping to send to the notifiables.</td></tr><tr><td>properties</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>A struct of properties to populate the notification with.</td></tr></tbody></table>

**Return**: `notification` instance, after it has been populated and sent to the `notifiables`.


## Durable queue

`queue(notification, properties={}, options={})` delegates to NotificationService with this notifiable and returns NotificationPublication. The released `notify(notification, properties={})` still returns BaseNotification.
