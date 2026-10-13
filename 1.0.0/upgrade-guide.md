# Upgrade Guide

If your application uses synchronous `notify()` calls, you can keep those calls when upgrading to 1.1.0. Notifications still extend `BaseNotification`, and `notify()` still returns the populated Notification instance. Existing numbered inbox cursors also remain available.

### Applying the migrations

Run the module migrations through cfmigrations and its migration ledger, using the same datasource as your domain writes. A database queue on that datasource can share the publication transaction. Keep previously applied migrations unchanged.

The additive `2026_10_12_180000_add_megaphone_notification_mapping.cfc` migration runs after the existing migrations. It adds a nullable `notificationMapping` column that tells the worker which Notification class to load.

Backfilling is required for older durable events whose deliveries you want the new dispatcher to send or recover. It is optional for completed, history-only events, and existing inbox entries can still be read without it. Fresh installations have nothing to backfill; new plans and newly queued events capture their mapping automatically.

For example, an event type such as `offer.published` is a business label. Its mapping might be `ActivityNotification`, the WireBox name of the class that can format that event for its saved channels. Without a mapping, eligible old work fails during preparation and cannot send. Use a class that understands the original payload, and update only the mapping: keep existing payloads, hashes, delivery IDs, and queue references. Backfilling does not recreate deliveries or resend completed notifications.

If you use custom table names, update the migration configuration to match your storage settings. Try the upgrade against a copy of existing data, including pending deliveries and saved plans, before switching your application to the new APIs.

### Queueing with cbq

Megaphone uses its bundled `CBQNotificationQueue` adapter by default, but cbq is an optional application dependency. Install cbq in your application to use that adapter. If cbq is unavailable when the adapter tries to enqueue, Megaphone throws `Megaphone.Queue.MissingCBQ`. Choose a queue on a registered cbq connection and follow that connection Provider's setup instructions. Database migrations are needed when you use cbq's database Provider. Keep your application's cbq configuration and add a notification worker when your Provider uses worker pools. See [Durable Delivery](guides/durable-delivery.md#setting-up-cbq) for setup.

You can keep an existing `queue.queueAdapter` override. If you previously left the adapter unset to poll saved deliveries, explicitly set it to an empty string to retain that behavior.

### Keeping the first read timestamp

`markAsRead(readDate)` and `markAllAsRead(readDate)` retain their released behavior: they replace the read timestamp. If your application wants to preserve the first read time, switch those calls to `markAsReadIfUnread()` and `markAllAsReadIfUnread()`.

See [Notification Inboxes](guides/notification-inboxes.md) for snapshots, archive state, and authorization constraints.
