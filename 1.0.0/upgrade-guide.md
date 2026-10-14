# Upgrade Guide

If your application uses synchronous `notify()` calls, you can keep those calls when upgrading to 1.1.0. Notifications still extend `BaseNotification`, and `notify()` still returns the populated Notification instance. Existing numbered inbox cursors also remain available.

### Applying the migrations

To use the new notification storage and delivery features, apply Megaphone's module migrations through cfmigrations and its migration ledger. Use the same datasource as your domain writes; a database queue on that datasource can share the publication transaction. Keep previously applied migrations unchanged.

If you use custom table names, update the migration configuration to match your storage settings. Test the migrations against a copy of your database before applying them to your deployed application.

### Queueing with cbq

Megaphone uses its bundled `CBQNotificationQueue` adapter by default, but cbq is an optional application dependency. Install cbq in your application to use that adapter. If cbq is unavailable when the adapter tries to enqueue, Megaphone throws `Megaphone.Queue.MissingCBQ`. Choose a queue on a registered cbq connection and follow that connection Provider's setup instructions. Database migrations are needed when you use cbq's database Provider. Keep your application's cbq configuration and add a notification worker when your Provider uses worker pools. See [Durable Delivery](guides/durable-delivery.md#setting-up-cbq) for setup.

### Keeping the first read timestamp

`markAsRead(readDate)` and `markAllAsRead(readDate)` retain their released behavior: they replace the read timestamp. If your application wants to preserve the first read time, switch those calls to `markAsReadIfUnread()` and `markAllAsReadIfUnread()`.

See [Notification Inboxes](guides/notification-inboxes.md) for snapshots, archive state, and authorization constraints.
