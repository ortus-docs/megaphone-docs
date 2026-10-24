# What's New?

### Unreleased

[Push registrations](guides/push-registrations.md#reading-exclusions-for-several-notification-types) adds `getExcludedIdsForTypes()` for notification settings pages. Read several types' device exclusions together and access the results by notification type. Recipient ownership and database comparisons match the scalar lookup. The existing bounded reader handles up to 100 distinct types per query; no schema migration is needed.

### 1.3.0

Digests can now send through any of your existing delivery channels. [DigestProvider](providers/digestprovider.md) collects items for a named schedule, then publishes another Notification. That Notification's `via()` chooses email, SMS, or other channels, and its Provider methods supply the content. One daily summary can send through multiple channels without collecting its items twice.

[DigestService](reference/digestservice.md) stores the user's subscriptions and adds `addSubscription()`, `updateSubscription()`, `stopSubscription()`, `resumeSubscription()`, and `transferSubscription()`. Schedule changes keep eligible pending items. Transfers let a user switch from daily to weekly while explicitly choosing whether to move or discard pending items. Started or uncertain deliveries are not republished at the destination.

The [Digests guide](guides/digests.md) walks through daily and weekly audition notifications for CommuniArts. It covers subscription ownership, independent delivery preferences, immediate summaries, selection, and transfer approval.

The 1.2.0 email digest API remains supported. Apply the additive publication migration before using 1.3.0, and see the [Upgrade Guide](upgrade-guide.md) for the difference between legacy subscription updates and the new lifecycle methods.

You can also [queue several Notifications](guides/durable-delivery.md#queueing-several-notifications) in one transaction with `planMany()` and `queuePlans()`. Producers that do not need result objects can use `admitPlans()` or `admitRequests()`. Existing queue adapters remain supported; adapters can opt into batch handoff.

### 1.2.0

Want to send a summary instead of a separate email for every Notification? [EmailDigestProvider](providers/emaildigestprovider.md) collects content for named digests. The module's ColdBox scheduler sends the summaries on each subscription's schedule. A Notifiable can subscribe to product updates and organization news independently. Immediate subscriptions use the same digest renderer with one item, so a product needs only the `emailDigest` route. Immediate-only definitions can omit their calendar schedule.

Your Notifications still choose channels through `via()`. Immediate email uses the ordinary email channel; Digest collects through `toEmailDigest()`. `notify()` and `queuePlan()` retain those conventions, and `NotificationPublication.getCollections()` keeps collection separate from actual email delivery.

[Named Email Digests](guides/email-digests.md) covers local-time schedules, consent and item eligibility, rendering summaries, automatic module scheduling, and unsubscribe links for one digest. `EmailDigestService` freezes stable batches and reuses Megaphone's existing receipt-aware email delivery, retry, and recovery APIs. Preference changes apply prospectively without replaying old collected items.

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
