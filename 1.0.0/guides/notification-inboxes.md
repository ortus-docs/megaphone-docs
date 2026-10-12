# Notification Inboxes

The [DatabaseProvider](../providers/databaseprovider.md) gives you an inbox of saved Notifications for a Notifiable. You can keep using numbered pages, or use a cursor for a feed with a "load more" button.

### Loading the next page

Inject `DatabaseNotificationService@megaphone` and call `getNotificationsAfter()` for the first page:

```js
var inbox = databaseNotificationService.getNotificationsAfter(
    notifiable = user,
    maxRows = 25
);
var notifications = inbox.getResults();
```

The returned [DatabaseNotificationKeysetCursor](../reference/baseprovider/databaseprovider/databasenotificationkeysetcursor.md) contains `DatabaseNotification` objects. Call `getMemento()` when returning the page from an HTTP endpoint. Its shape is `{ results, nextCursor, hasMore }`.

For another request, pass the returned token as `afterCursor`:

```js
var inbox = databaseNotificationService.getNotificationsAfter(
    notifiable = user,
    afterCursor = nextCursor,
    maxRows = 25
);
```

If you are iterating in the same request, check `hasNext()` and call `next()` to advance the cursor. The [HasDatabaseNotifications delegate](../reference/baseprovider/databaseprovider/hasdatabasenotifications.md) provides the same methods directly on your Notifiable.

Cursor pages are ordered by the exact stored creation time, then ID, newest first. New arrivals won't shift an existing boundary, and the cursor still works if the boundary notification is deleted. Tokens are bound to the Notifiable, so a token from another user's inbox is rejected. The existing numbered-page cursor APIs are still available.

### Filtering an inbox

`getNotificationsAfter()` also accepts `channelName` (default `database`), `constraints`, `archiveMode` (default `active`), and `unread` (default `false`). Reset the cursor when you change filters.

For archive and group queries, apply the inbox-state migration and enable `inboxState=true` in the Database channel's `properties`.

Use `constraints` for application authorization, such as limiting notifications to records the user can still access. Megaphone applies them together with ownership checks before pagination and keeps them on loaded Notifications for later mutations.

Display filters have a different purpose. For example, reading a notification removes it from an unread view, but shouldn't make its already-authorized handle unusable. On a numbered cursor, `configureQuery(callback, constrainMutations=false)` lets you add display filters without making them mutation constraints. Keep authorization in `constraints`.

### Marking notifications as read

`markAsRead(readDate)` replaces the stored read timestamp, even when the Notification has already been read. The numbered cursor's `markAllAsRead(readDate)` does the same for its matching entries.

If you want to keep the first read time, use the `IfUnread` variants:

```js
notifications[ 1 ].markAsReadIfUnread();
user.getNotifications().markAllAsReadIfUnread();
```

For a "mark what I just saw as read" action, capture IDs with `snapshotIds()` and pass those IDs to `markSnapshotAsRead()`. Megaphone only updates captured IDs that are still authorized and unread. Notifications arriving after the snapshot stay unread.

`archive()` and `unarchive()` change display state without changing the read date. Ownership and authorization checks still apply.

### Database configuration

Your existing database adapter can configure `cursorTimestampExpression`, `cursorTimestampSqlType`, and `idSqlType` on the channel. The timestamp expression must preserve the stored precision; PostgreSQL UUID columns use `other` for the ID SQL type. Set these in server configuration, where they match your schema.

### Importing and retaining history

Use `importNotification()` to bring existing inbox history into Megaphone with stable IDs. It stores the history without sending a notification.

`pruneNotifications()` removes bounded batches above your configured inbox cap. Inbox retention is separate from event replay and delivery evidence retention, so you can choose how much visible history to keep without using the inbox as a delivery ledger.
