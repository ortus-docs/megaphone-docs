# Upgrade Guide

## Upgrading to 1.3.0

The 1.2.0 email digest API still works. Existing `EmailDigestProvider` channels, `toEmailDigest()`, renderers, `emailChannel`, and `setSubscription()` calls keep their behavior. You can adopt the new API for a new digest without migrating existing email definitions.

Apply `2026_10_18_120000_add_generic_digest_publications` after the existing digest migration. It adds a saved `resumeMode` to subscriptions and a publication table linking batches to summary events. Existing subscriptions, admissions, pending batches, and delivery history are preserved. Keep previously applied migrations unchanged.

Digest storage remains under `storage.digests` and must share the transactional datasource used by events and deliveries. The new table defaults to `megaphone_digest_publications`; configure `storage.digests.publicationsTable` when using another name. Custom table names need equivalent application migrations.

For a provider-independent digest, register a [DigestProvider](providers/digestprovider.md) channel with `properties.digestName`. Replace the definition's delivery channel and renderer with a summary `notification` mapping. The source Notification supplies `toDigest()`; the summary Notification supplies its own `via()` and Provider formatters. Keep legacy definitions in their existing format.

The new lifecycle methods use explicit subscription activation. A missing subscription does not collect items. `updateSubscription()` preserves eligible pending items, while legacy `setSubscription()` retains its existing invalidation behavior. Stop and resume apply prospectively and do not replay suppressed items. Use `transferSubscription()` to replace a named daily subscription with a weekly one, choosing `pendingItems="transfer"` or `"discard"` explicitly.

A stopped subscription created before 1.3.0 has no saved previous mode, so its new `resumeMode` defaults to `digest`. If it should resume in immediate mode, call `updateSubscription(mode="immediate")` with its Notifiable and digest name before resuming it.

See [Digests](guides/digests.md) for the complete setup and examples.

## Upgrading to 1.1.0

If your application uses synchronous `notify()` calls, you can keep those calls when upgrading to 1.1.0. Notifications still extend `BaseNotification`, and `notify()` still returns the populated Notification instance. Existing numbered inbox cursors also remain available.

### Applying the migrations

To use the new notification storage and delivery features, apply Megaphone's module migrations through cfmigrations and its migration ledger. Use the same datasource as your domain writes; a database queue on that datasource can share the publication transaction. Keep previously applied migrations unchanged.

If you use custom table names, update the migration configuration to match your storage settings. Test the migrations against a copy of your database before applying them to your deployed application.

### Queueing with cbq

Megaphone uses its bundled `CBQNotificationQueue` adapter by default, but cbq is an optional application dependency. Install cbq in your application to use that adapter. If cbq is unavailable when the adapter tries to enqueue, Megaphone throws `Megaphone.Queue.MissingCBQ`. Choose a queue on a registered cbq connection and follow that connection Provider's setup instructions. Database migrations are needed when you use cbq's database Provider. Keep your application's cbq configuration and add a notification worker when your Provider uses worker pools. See [Durable Delivery](guides/durable-delivery.md#setting-up-cbq) for setup.

### Keeping the first read timestamp

`markAsRead(readDate)` and `markAllAsRead(readDate)` retain their released behavior: they replace the read timestamp. If your application wants to preserve the first read time, switch those calls to `markAsReadIfUnread()` and `markAllAsReadIfUnread()`.

See [Notification Inboxes](guides/notification-inboxes.md) for snapshots, archive state, and authorization constraints.
