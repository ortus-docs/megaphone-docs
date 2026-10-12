# Upgrade Guide

If your application uses synchronous `notify()` calls, you can keep those calls when upgrading to 1.1.0. Notifications still extend `BaseNotification`, and `notify()` still returns the populated Notification instance. Existing numbered inbox cursors also remain available.

### Applying the migrations

Run the module migrations through cfmigrations and its migration ledger, using the same datasource as your domain writes and queue jobs. Keep previously applied migrations unchanged.

The additive `2026_10_12_180000_add_megaphone_notification_mapping.cfc` migration runs after the existing migrations. It adds a nullable mapping so workers can reconstruct Notifications for stored events. Backfill the mapping for known notification types in your application. Keep their existing payloads, hashes, delivery IDs, and queue references.

If you use custom table names, update the migration configuration to match your storage settings. Try the upgrade against a copy of existing data, including pending deliveries and saved plans, before switching your application to the new APIs.

### Moving from the unreleased durable APIs

If you adopted an earlier 1.1.0 candidate, update these calls:

| Earlier API | Use instead |
| --- | --- |
| `DurableNotificationService.publish()` | `NotificationService.plan()`, `queue()`, and `queuePlan()` |
| `DeliveryDispatcher` | `NotificationDeliveryService.dispatch()` |
| `getNotificationSlice()` | `getNotificationsAfter()`, returning a `DatabaseNotificationKeysetCursor` |

You now define durable notifications using the same Notification, Notifiable, Provider, and delegate patterns as synchronous delivery. See [Durable Delivery](guides/durable-delivery.md) for the queue and Provider setup. Use the public services for delivery operations, preferences, and subscriptions; stores handle persistence behind those services.

If your application saved admission documents using the earlier shape, add a reader that translates those records into the version 1 `NotificationPlan` memento. Preserve the original recipients, channel choices, device IDs, and occurrence time. Reconstruct the saved decision instead of calling `plan()` again against today's audience and preferences.

Keep your existing snapshot hash algorithm configured when replaying old events. If you already have encrypted push subscriptions, retain their encryption contexts too. The [Event Identities](guides/event-identities.md) and [Push Registrations](guides/push-registrations.md) guides cover those settings.

### Keeping the first read timestamp

`markAsRead(readDate)` and `markAllAsRead(readDate)` retain their released behavior: they replace the read timestamp. If your application wants to preserve the first read time, switch those calls to `markAsReadIfUnread()` and `markAllAsReadIfUnread()`.

See [Notification Inboxes](guides/notification-inboxes.md) for snapshots, archive state, and authorization constraints.
