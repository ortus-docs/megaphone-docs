# Notification Inboxes

The [DatabaseProvider](../providers/databaseprovider.md) gives you an inbox of saved Notifications for a Notifiable. You can keep using numbered pages, or use a cursor for a feed with a "load more" button.

### Choosing a pagination method

Both methods belong to the existing `DatabaseNotificationService` and return cursor objects. Choose the one that matches your interface:

| Method | Use it for | Navigation |
| --- | --- | --- |
| `getNotifications()` | Numbered pages and views that show totals. | Start at `initialPage`, move with `next()` or `previous()`, and read `getPagination()` for totals. |
| `getNotificationsAfter()` | A feed with "load more" or infinite scrolling. | Continue from an `afterCursor` token or call `next()`. No previous-page method, page numbers, or totals. |

Suppose a user loads ten Notifications, then three new ones arrive. With numbered pages, those arrivals shift the positions used for page two, so some entries from page one can appear again. `getNotificationsAfter()` continues from the last seen creation time and ID instead, making it a better fit for a feed that changes while the user browses.

`getNotificationsAfter()` is an additional option; `getNotifications()` remains supported. Their archive defaults also differ: numbered pages include all entries by default, while `getNotificationsAfter()` defaults to active entries. Pass `archiveMode="all"` if you want the latter to include archived entries too.

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

Loaded Notifications keep a separate query for mutations containing ownership and application authorization rules. It leaves out the page boundary and read/archive display filters. An entry loaded from an active, unread view can therefore be marked read, archived, and restored through the same handle. A real authorization change still prevents that handle from changing the entry.

### Marking notifications as read

`markAsRead(readDate)` replaces the stored read timestamp, even when the Notification has already been read. The numbered cursor's `markAllAsRead(readDate)` does the same for its matching entries.

If you want to keep the first read time, use the `IfUnread` variants:

```js
notifications[ 1 ].markAsReadIfUnread();
user.getNotifications().markAllAsReadIfUnread();
```

### Marking a captured set as read

Use `snapshotIds()` when an action should apply to the notifications matching the inbox at the start of that action. It captures all matching, authorized IDs, without a page limit:

```js
var capturedIds = databaseNotificationService.snapshotIds(
    notifiable = user,
    archiveMode = "active"
);

// Another Notification may arrive before this operation finishes.
databaseNotificationService.markSnapshotAsRead(
    notifiable = user,
    ids = capturedIds
);
```

Notifications arriving after the snapshot stay unread. `markSnapshotAsRead()` rechecks ownership and your authorization `constraints`, and only updates unread entries, preserving their first read timestamp. Pass your current authorization constraints when capturing and marking the set.

`snapshotIds()` captures the whole matching inbox, rather than just the displayed page. For a "mark what I just saw as read" action, take IDs from the page's results instead:

```js
var displayedIds = inbox.getResults().map( ( notification ) => notification.getId() );
databaseNotificationService.markSnapshotAsRead(
    notifiable = user,
    ids = displayedIds
);
```

`archive()` and `unarchive()` change display state without changing the read date. Ownership and authorization checks still apply.

### Database configuration

Your existing database adapter can configure `cursorTimestampExpression`, `cursorTimestampSqlType`, and `idSqlType` on the channel. The timestamp expression must preserve the stored precision; PostgreSQL UUID columns use `other` for the ID SQL type. Set these in server configuration, where they match your schema.

### Importing and retaining history

Use `importNotification()` to bring existing inbox history into Megaphone with stable IDs. It stores the history without sending a notification.

`pruneNotifications()` removes bounded batches above your configured inbox cap. Inbox retention is separate from event replay and delivery evidence retention, so you can choose how much visible history to keep without using the inbox as a delivery ledger.
