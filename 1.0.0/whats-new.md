# What's New?

### 1.2.0

Want to send a summary instead of a separate email for every Notification? [EmailDigestProvider](providers/emaildigestprovider.md) collects content for named digests. `EmailDigestService` sends the summaries on each subscription's schedule. A Notifiable can subscribe to product updates and organization news independently.

Your Notifications still choose channels through `via()`. Immediate email uses the ordinary email channel; Digest collects through `toEmailDigest()`. `notify()` and `queuePlan()` retain those conventions, and `NotificationPublication.getCollections()` keeps collection separate from actual email delivery.

[Named Email Digests](guides/email-digests.md) covers local-time schedules, consent and item eligibility, rendering summaries, scheduler calls, and unsubscribe links for one digest. `EmailDigestService` freezes stable batches and reuses Megaphone's existing receipt-aware email delivery, retry, and recovery APIs. Preference changes apply prospectively without replaying old collected items.

[Delivery Operations](guides/delivery-operations.md#reconciling-a-confirmed-rejection) also adds `reconcileNotAccepted()`. Use authoritative rejection evidence to release an ambiguous delivery for another bounded attempt, while keeping its transport counts. Unknown outcomes continue to require reconciliation rather than an automatic resend.

Apply the additive digest migration and configure `digests` and `storage.digests` before using collection. Existing synchronous Notifications and ordinary immediate email channels continue to work.

### 1.1.0

You can now queue a Notification using the same conventions as `notify()`. Pass a Notifiable, a Notification instance or WireBox name, and its properties. Megaphone saves the delivery decisions so a worker can send it later.

Megaphone includes a default cbq adapter and expects your application to install cbq when using it, so you don't need to write your own delivery job. You can still configure another adapter when your application needs one.

Want to capture the decision before handing work to a queue? `plan()` returns a `NotificationPlan` you can save and restore. Repeating a stable event key returns the original work, without adding recipients or channels that weren't in the first plan.

There are also new ways to manage notifications in your application:

- [Notification Inboxes](guides/notification-inboxes.md) adds "load more" pagination, archive state, and read snapshots.
- [Notification Preferences](guides/notification-preferences.md) stores channel choices for scopes defined by your application.
- [WebPushProvider](providers/webpushprovider.md) sends through registered devices, with [Push Registrations](guides/push-registrations.md) for device management.
- [Delivery Operations](guides/delivery-operations.md) lets you inspect history, reconcile receipts, and request recovery.

Your existing synchronous notifications and numbered inbox cursors still work. Start with [Durable Delivery](guides/durable-delivery.md), or read the [Upgrade Guide](upgrade-guide.md) if you are moving an existing application to 1.1.0.
