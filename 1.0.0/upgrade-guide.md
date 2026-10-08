# Upgrade Guide

## Upgrading to 1.1.0

Your existing notification and provider APIs still work. The new storage features need migrations and application configuration. Installing the module doesn't import your existing history, preferences, or pending deliveries.

1. Check that 1.1.0 has been published on [ForgeBox](https://forgebox.io/view/megaphone). The candidate is currently in [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2).
2. Apply the migrations you need, in order, to the same datasource as your application transaction. Keep the original notifications table migration if you're installing that provider for the first time.
3. Configure recipient access constraints, preference scopes, encryption keys, and any queue and delivery callbacks your app uses.
4. Import history with stable IDs and plan the handoff from your old delivery owner before enabling the new worker.

### Migrations

The new files are under `resources/database/migrations` in the module:

- `2026_10_05_180000_add_megaphone_inbox_state.cfc`
- `2026_10_05_180001_create_megaphone_events.cfc`
- `2026_10_05_180002_create_megaphone_deliveries.cfc`
- `2026_10_05_180003_create_megaphone_delivery_attempts.cfc`
- `2026_10_05_180004_create_megaphone_preferences.cfc`
- `2026_10_05_180005_create_megaphone_subscriptions.cfc`
- `2026_10_05_180006_create_megaphone_device_exclusions.cfc`
- `2026_10_06_010000_add_megaphone_recovery_allowance.cfc`
- `2026_10_06_010001_create_megaphone_delivery_recoveries.cfc`
- `2026_10_06_010002_complete_megaphone_recovery_allowance.cfc`
- `2026_10_08_210000_preserve_megaphone_delivery_timestamp_precision.cfc`

Events come before deliveries; deliveries come before attempts and recovery receipts. Run the related migrations together for each feature. The final migration preserves fractional seconds in MySQL delivery timestamps; it leaves other database grammars alone.

For [inbox archive state and grouping](guides/notification-inboxes.md), apply the inbox-state migration and set `properties.inboxState = true` on the database channel. If you use a custom `properties.table`, apply those column and index changes to that table. Durable stores can also use custom table names; adapt their migrations to match your configuration.

### Database adapters

The database channel's table setting is `properties.table`. For PostgreSQL UUID IDs, set `idSqlType = "other"`. [Keyset pages](guides/notification-inboxes.md#keyset-inbox-pages) also require a timestamp expression that preserves database precision and the appropriate JDBC binding type. Offset pages don't require that configuration.

### Bringing existing data over

[History imports](guides/notification-inboxes.md#importing-existing-notifications) don't send notifications. Keep stable notification IDs, track imported preferences, and retain the source delivery's attempt and acceptance evidence.

Stop the old delivery owner before importing pending work. Queued imports can become due after commit. Coordinate the worker handoff so the old and new systems can't both send the same delivery. Import accepted work with verified provider evidence; uncertain acceptance stays ambiguous. Plan rollback so it won't resend accepted work.

Choose your inbox cap separately from the event replay window and delivery evidence retention. Use [replay cutoffs and cleanup callbacks](guides/delivery-operations.md#limiting-event-replay) to keep external references and retirement receipts consistent.

### Application configuration

Your app chooses notification content, destinations, authorization, preference scopes, and which channels are enabled. Durable delivery also needs eligibility and sender callbacks, a due-work scheduler, expired-lease recovery, and an optional transactional queue adapter. [The delivery guide](guides/durable-delivery.md) covers those boundaries.

Web push needs subscription encryption keys, VAPID credentials, trusted push-service hosts, Java 11+, and [the optional SDK](guides/web-push.md). You'll also need browser enrollment and service-worker code in your app. Installing Megaphone doesn't set those up for you.

You can decide separately whether Megaphone handles authentication and account-access mail. Test your actual providers and devices before switching delivery ownership.
