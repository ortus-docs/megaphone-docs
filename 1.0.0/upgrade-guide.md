# Upgrade guide

The released synchronous API remains compatible. Read timestamp replacement is retained; use the new IfUnread variants when preserving the first read time.

Apply `2026_10_12_180000_add_megaphone_notification_mapping.cfc` after the existing migrations. Backfill reconstruction mappings for existing events without changing payloads, hashes, delivery IDs, or queue references. Application readers of saved RC admissions must translate their old field names into the version 1 plan shape without resolving a new audience or preferences.

The unreleased `DurableNotificationService`, `DeliveryDispatcher`, and struct-returning `getNotificationSlice()` API have been replaced. Stores remain persistence implementation details; consumers use the services above. Merging, package publication, consumer installation, and deployment are separate verification steps.

Apply all module migrations through cfmigrations and its ledger, on the same datasource as domain writes and queue jobs. Keep historical migrations unchanged. Test an upgrade with pending work and saved admissions before switching consumers.
