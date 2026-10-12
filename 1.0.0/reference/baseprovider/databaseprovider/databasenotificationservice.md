# DatabaseNotificationService

The 1.1.0 inbox APIs add constrained retrieval, unread counts, snapshots, imports, pruning, and keyset slices. See [notification inboxes](../../../guides/notification-inboxes.md) for configuration and examples.


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

## Inbox cursors and read state

`DatabaseNotificationService.getNotificationsAfter(notifiable, channelName="database", afterCursor="", maxRows=25, constraints, archiveMode="active", unread=false)` returns `DatabaseNotificationKeysetCursor`. The delegate offers the same method with an implicit notifiable. Use `getResults()`, `getNextCursor()`, `hasNext()`, and `next()`; its memento is `{ results, nextCursor, hasMore }`.

Keep existing numbered-page cursor methods for offset pagination. Keyset pagination orders by exact stored creation time and ID, handles deleted boundaries and concurrent arrivals, and binds tokens to the notifiable. Configure `cursorTimestampExpression`, `cursorTimestampSqlType`, and `idSqlType` for the existing database adapter. PostgreSQL UUID columns use `other`; timestamp text must preserve precision. These settings are backend configuration, never client input.

Apply the inbox-state migration and enable `properties.inboxState=true` for archive/group queries. Constraints run inside ownership filters before pagination and remain on loaded handles for mutations. `configureQuery(callback, constrainMutations=false)` is only for display filters; leave authorization constraints enabled. Reset the cursor when filters change.

Released `markAsRead(readDate)` and cursor `markAllAsRead(readDate)` replace the timestamp. Use `markAsReadIfUnread()` and `markAllAsReadIfUnread()` to preserve the first read time. Read snapshots update only captured, currently authorized IDs; later arrivals remain unread. Archive and unarchive leave read state alone.

`importNotification()` retains stable history IDs and sends nothing. `pruneNotifications()` removes bounded batches above the configured inbox cap. Inbox retention is separate from event replay and delivery evidence retention.
