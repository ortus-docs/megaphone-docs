## Inbox cursors and read state

`DatabaseNotificationService.getNotificationsAfter(notifiable, channelName="database", afterCursor="", maxRows=25, constraints, archiveMode="active", unread=false)` returns `DatabaseNotificationKeysetCursor`. The delegate offers the same method with an implicit notifiable. Use `getResults()`, `getNextCursor()`, `hasNext()`, and `next()`; its memento is `{ results, nextCursor, hasMore }`.

Keep existing numbered-page cursor methods for offset pagination. Keyset pagination orders by exact stored creation time and ID, handles deleted boundaries and concurrent arrivals, and binds tokens to the notifiable. Configure `cursorTimestampExpression`, `cursorTimestampSqlType`, and `idSqlType` for the existing database adapter. PostgreSQL UUID columns use `other`; timestamp text must preserve precision. These settings are backend configuration, never client input.

Apply the inbox-state migration and enable `properties.inboxState=true` for archive/group queries. Constraints run inside ownership filters before pagination and remain on loaded handles for mutations. `configureQuery(callback, constrainMutations=false)` is only for display filters; leave authorization constraints enabled. Reset the cursor when filters change.

Released `markAsRead(readDate)` and cursor `markAllAsRead(readDate)` replace the timestamp. Use `markAsReadIfUnread()` and `markAllAsReadIfUnread()` to preserve the first read time. Read snapshots update only captured, currently authorized IDs; later arrivals remain unread. Archive and unarchive leave read state alone.

`importNotification()` retains stable history IDs and sends nothing. `pruneNotifications()` removes bounded batches above the configured inbox cap. Inbox retention is separate from event replay and delivery evidence retention.
