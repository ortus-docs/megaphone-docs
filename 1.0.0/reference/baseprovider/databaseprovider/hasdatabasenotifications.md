# HasDatabaseNotifications

This is a [delegate component](https://wirebox.ortusbooks.com/usage/wirebox-delegators) and requires ColdBox 7+. It should be added to an [`INotifiable`](../../inotifiable.md) instance.

### getNotifications

Returns all database notifications for this Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of notifications for this notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getReadNotifications

Returns all read database notifications for this Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve read notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of read notifications for this notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getUnreadNotifications

Returns all unread database notifications for this Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve unread notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of unread notifications for this notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getNotificationsAfter

Loads a cursor page from this Notifiable's inbox. The delegate supplies the Notifiable for you:

```js
var inbox = user.getNotificationsAfter( maxRows = 25 );
```

It accepts `channelName`, `afterCursor`, `maxRows`, `constraints`, `archiveMode`, and `unread`, with the same defaults as [DatabaseNotificationService.getNotificationsAfter](databasenotificationservice.md#getnotificationsafter). Loaded Notifications keep ownership and authorization checks for later mutations.

**Return**: A [DatabaseNotificationKeysetCursor](databasenotificationkeysetcursor.md).

### Other inbox methods

The delegate also provides `getNotification()`, `countUnreadNotifications()`, `snapshotIds()`, `markSnapshotAsRead()`, and `pruneNotifications()`. Each uses this Notifiable, so you can call the service's inbox operations directly from your user or other Notifiable model.

See [Notification Inboxes](../../../guides/notification-inboxes.md) for examples and read-state behavior.
