# Notification inboxes

You can use `DatabaseNotificationService@megaphone` to look up a recipient's
notifications, count unread items, mark a snapshot as read, import history, and
clean up old entries. The existing retrieval arguments still work. If you use the
`HasDatabaseNotifications` delegate, these methods use the parent object as the
recipient.

To add archive state and grouping, apply the inbox-state migration and set
`properties.inboxState = true` on your database channel. The migration adds
nullable `archivedDate` and `groupKey` columns and indexes for recipient, order,
and group lookups. Existing channels keep using the original schema by default.
If you use a custom `properties.table`, apply the same changes to that table.
For PostgreSQL UUID IDs, set `properties.idSqlType = "other"`. The default string
binding still works for string ID columns.

```cfc
var inbox = wirebox.getInstance( "DatabaseNotificationService@megaphone" );
var visible = ( query ) => query.whereIn( "groupKey", authorizedGroupKeys );
var page = inbox.getNotifications(
    notifiable = currentUser,
    constraints = visible,
    archiveMode = "active",
    initialPage = 1,
    maxRows = 25
);
var unread = inbox.countUnreadNotifications(
    notifiable = currentUser, constraints = visible, archiveMode = "active"
);
var ids = inbox.snapshotIds(
    notifiable = currentUser, constraints = visible, archiveMode = "active"
);
inbox.markSnapshotAsRead( notifiable = currentUser, ids = ids, constraints = visible );
```

The `constraints` callback runs inside the recipient filter, before pagination
and counting. Loaded notifications keep that constraint for later updates.
`getNotification` returns null if the ID is missing, belongs to another recipient,
or fails your constraint. You still need to supply your application's current
authorization rules; a `groupKey` by itself doesn't grant access.

`archiveMode` accepts `all` (the existing default), `active`, or `archived`.
Calling `archive()` or `unarchive()` leaves the read date alone. Archived entries
stay in the database, so you can restore them later. Changing read or archive
state doesn't invalidate an already loaded notification just because it no
longer matches the page's display filter. Recipient and visibility checks still
apply when you update it or refresh its state.

Cursor `configureQuery` callbacks apply to both retrieval and updates by default.
Use them to configure the supplied query, without calling external services.
Set `constrainMutations=false` only for display filters that shouldn't limit later
updates. Keep the default for authorization constraints.

A read snapshot contains the IDs visible when you took it. Check authorization
again when marking those IDs as read. Notifications that arrive afterward stay
unread, and marking an item as read again preserves its original read date.
The existing cursor-wide methods are still available; use snapshots when you
want to leave new arrivals alone.

### Importing existing notifications

`importNotification(notifiable, id, type, data, createdDate, readDate, channelName,
groupKey, archivedDate)` saves history without sending anything. Running it again
leaves the existing row alone. An ID belonging to another recipient or type is
rejected. Check your source IDs and payloads before importing, and call this
method inside your import transaction.

`PreferenceStore.importChoice(notifiable, notificationType, scopeKey, channel,
choice)` adds a missing explicit choice without replacing one that's already
there, even when two imports run at once. `inherit` creates no row. Keep a record
of which choices you've imported so rerunning a backfill doesn't recreate a
choice the user has since changed to Inherit. Importing choices sends nothing.

`DeliveryStore.importDelivery(intent, history)` imports delivery state without
calling a provider. `intent` uses the same identity and routing fields as
`enqueue`; `history` supplies state, attemptCount, createdDate, and optional
settledDate, reason, and providerReference. To import accepted work, supply a
bounded reference to evidence that it was accepted. Ambiguous work stays out of
automatic retries. Repeated imports preserve the current state and don't invent
attempts.

Before importing deliveries, verify the source evidence, stop the old delivery
owner, and keep its attempt and audit references. Queued work becomes eligible
for recovery after commit, so coordinate that import with the handoff to your
new worker.

`pruneNotifications(notifiable, keep=1000, batchSize=200, channelName="database")`
removes one batch of the oldest entries above the cap, including archived entries.
Call it on a schedule until the excess is gone. It only removes inbox entries;
domain history, delivery attempts, and event identities have their own retention.


## Keyset inbox pages

`DatabaseNotificationService.getNotificationSlice()` returns
`{ results, nextCursor, hasMore }`. Pass `nextCursor` as `afterCursor` to get the
next page. Results are ordered by newest `createdDate`, then descending ID.
New entries ahead of that position won't repeat items, and deleting the boundary
row won't break the next page.

The cursor belongs to a recipient, but you'll still need to check access.
Recipient ownership, visibility constraints, archive mode, and unread filters
apply on every page. Reset the cursor when filters change. Start a fresh first
page to include new arrivals.

Configure the database provider's `cursorTimestampExpression` to return timestamp
text as `yyyy-MM-dd HH:mm:ss[.fraction]` without losing database precision.
Set `cursorTimestampSqlType` to the JDBC binding type your adapter needs.
For PostgreSQL, use `cursorTimestampExpression = 'CAST("createdDate" AS TEXT)'`
and `cursorTimestampSqlType = "other"`. UUID IDs also need `idSqlType = "other"`.
Keep these expressions in application configuration, never caller input.
The existing offset pagination methods don't need these settings.

An invalid cursor or one from another recipient throws
`Megaphone.Database.InvalidCursor`. Set `cursorIdentifierPattern` for constrained
IDs, such as UUID columns, to reject malformed IDs before JDBC conversion.
You can keep previous cursor positions for back navigation, but each request
checks access again. A cursor keeps your place; it doesn't freeze permissions,
read state, or archive state. Pruned entries disappear without shifting a numeric
offset. Inbox pagination and retention leave pending deliveries and event
identities alone.
