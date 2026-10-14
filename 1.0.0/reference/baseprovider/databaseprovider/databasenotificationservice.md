# DatabaseNotificationService

Use this service to load a Notifiable's inbox, count unread Notifications, and manage saved history. For a walkthrough of pagination, read state, and authorization, see [Notification Inboxes](../../../guides/notification-inboxes.md).

### getNotifications

Returns all database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getReadNotifications

Returns all read database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve read notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve read notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of read notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getUnreadNotifications

Returns all unread database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve unread notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve unread notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of unread notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getNotificationsAfter

Returns a page of Notifications for a Notifiable using a forward-only cursor. Use it for an inbox with a "load more" button.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The Notifiable whose inbox to load. |
| `channelName` | `string` | false | `database` | The Database channel to query. |
| `afterCursor` | `string` | false | Empty string | A token from the previous page. Leave empty for the first page. |
| `maxRows` | `numeric` | false | `25` | The number of Notifications per page. |
| `constraints` | Query callback | false | Not set | Your application's authorization constraints. |
| `archiveMode` | `string` | false | `active` | Which archive state to display: `active`, `archived`, or `all`. |
| `unread` | `boolean` | false | `false` | Limits the page to unread Notifications when true. |

Notifications are ordered by exact creation time and ID, newest first. Cursor tokens belong to the Notifiable, and loaded Notifications retain authorization constraints for mutations. Reset the cursor when changing filters.

**Return**: A [DatabaseNotificationKeysetCursor](databasenotificationkeysetcursor.md).

### snapshotIds

Captures all IDs currently matching the Notifiable's ownership, `constraints`, and `archiveMode`, without a page limit. Pass `notifiable`, an optional `channelName`, authorization `constraints`, and `archiveMode` (default `all`). For just the displayed page, take IDs from the cursor's `getResults()` instead.

**Return**: An array of Notification IDs.

### markSnapshotAsRead

Marks captured IDs as read if they are still authorized and unread. Pass `notifiable`, the captured `ids`, and the same channel and authorization constraints. You can also pass `readDate`, which defaults to now.

Notifications that arrived after the snapshot stay unread. See [Notification Inboxes](../../../guides/notification-inboxes.md#marking-a-captured-set-as-read) for choosing between snapshots and the cursor's read methods.

**Return**: `void`.

### importNotification

Stores existing inbox history using a stable `id`, without sending through other channels. Pass `notifiable`, `id`, `type`, and a `data` struct. You can also preserve `createdDate`, `readDate`, `groupKey`, and `archivedDate`, and select a `channelName`.

Group and archive values require inbox state to be enabled. An ID already owned by another Notifiable or Notification type is rejected.

**Return**: The stored `DatabaseNotification` instance.

### pruneNotifications

Removes a bounded batch of old Notifications for a Notifiable, including archived entries. `keep` defaults to `1000`, `batchSize` to `200`, and `channelName` to `database`.

**Return**: The number of deleted Notifications.
