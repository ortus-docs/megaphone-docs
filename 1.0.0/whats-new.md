# What's New?

### 1.1.0

You can now queue a Notification using the same conventions as `notify()`. Pass a Notifiable, a Notification instance or WireBox name, and its properties. Megaphone saves the delivery decisions so a worker can send it later.

Megaphone includes cbq and a default queue adapter, so you don't need to write your own delivery job. You can still configure another adapter when your application needs one.

Want to capture the decision before handing work to a queue? `plan()` returns a `NotificationPlan` you can save and restore. Repeating a stable event key returns the original work, without adding recipients or channels that weren't in the first plan.

There are also new ways to manage notifications in your application:

- [Notification Inboxes](guides/notification-inboxes.md) adds "load more" pagination, archive state, and read snapshots.
- [Notification Preferences](guides/notification-preferences.md) stores channel choices for scopes defined by your application.
- [Java Web Push](guides/web-push.md) sends through registered devices, with [Push Registrations](guides/push-registrations.md) for device management.
- [Delivery Operations](guides/delivery-operations.md) lets you inspect history, reconcile receipts, and request recovery.

Your existing synchronous notifications and numbered inbox cursors still work. Start with [Durable Delivery](guides/durable-delivery.md), or read the [Upgrade Guide](upgrade-guide.md) if you are moving an existing application to 1.1.0.
