# Upgrade Guide

## Upgrading to 1.3.0

The 1.2.0 email digest API is deprecated but still works. Existing `EmailDigestProvider` channels, `toEmailDigest()`, renderers, `emailChannel`, and `setSubscription()` calls keep their behavior. You can adopt the new API for a new digest without migrating existing email definitions.

Apply `2026_10_18_120000_add_generic_digest_publications` after the existing digest migration. It adds a saved `resumeMode` to subscriptions and a publication table linking batches to summary events. Existing subscriptions, admissions, pending batches, and delivery history are preserved. Keep previously applied migrations unchanged.

Digest storage remains under `storage.digests` and must share the transactional datasource used by events and deliveries. The new table defaults to `megaphone_digest_publications`; configure `storage.digests.publicationsTable` when using another name. Custom table names need equivalent application migrations.

For a provider-independent digest, register a [DigestProvider](providers/digestprovider.md) channel with `properties.digestName`. Replace the definition's delivery channel and renderer with a summary `notification` mapping. The source Notification supplies `toDigest()`; the summary Notification supplies its own `via()` and Provider formatters. Keep legacy definitions in their existing format.

### Moving an email digest to DigestProvider

Keep the digest name, schedule, policy, and `storage.digests` settings. Change the collection Provider, move the digest name into its properties, and replace the renderer and email channel with a summary Notification mapping:

```diff
 moduleSettings.megaphone.channels.productUpdates = {
-    "provider": "EmailDigestProvider@megaphone",
-    "properties": {}
+    "provider": "DigestProvider@megaphone",
+    "properties": { "digestName": "productUpdates" }
 };

 moduleSettings.megaphone.digests.productUpdates = {
     "schedule": "everyWeekOn 1 09:00",
     "timezone": "UTC",
     "policy": "ProductDigestPolicy",
-    "emailChannel": "email",
-    "renderer": "ProductDigestRenderer"
+    "notification": "ProductDigestNotification"
 };
```

The collection channel can keep its existing name, so the source Notification's `via()` does not need to change. Rename its formatter to `toDigest()` and omit the digest name from the returned item:

```diff
-public struct function toEmailDigest( required any notifiable ) {
+public struct function toDigest( required any notifiable ) {
     return {
-        "digestName": "productUpdates",
         "itemKey": variables.digests.itemKey( [ "product.published", variables.publicationEventId, 1 ] ),
         "occurredAt": variables.digests.occurredAt( variables.publishedAt ),
         "data": { "productId": variables.productId }
     };
 }
```

Use `DigestService@megaphone` for new code. Its `itemKey()` and `occurredAt()` helpers keep the same format as `EmailDigestService@megaphone`.

Move the renderer's content preparation into `ProductDigestNotification`. Megaphone populates `batchId`, `digestName`, `cutoffAt`, and the selected `items` before routing the summary. It does not call the legacy renderer or populate renderer-specific properties such as `products` or `unsubscribeUrl`. Load current content and generate links in the summary's formatters, using the selected `items` as input.

Unlike the old renderer's returned Notification, the summary must implement `via()`:

```js
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="batchId";
    property name="digestName";
    property name="cutoffAt";
    property name="items";
    property name="productViews" inject="ProductDigestViews";

    public array function via( required any notifiable ) {
        return [ "email" ];
    }

    public any function toEmail( required any notifiable, required any newMail ) {
        return arguments.newMail(
            to = arguments.notifiable.getEmail(),
            from = "updates@example.com",
            subject = "Your product updates",
            type = "html"
        ).setBody( variables.productViews.email( variables.items, arguments.notifiable ) );
    }

}
```

`ProductDigestViews.email(items, notifiable)` is an application-specific rendering helper. Adapt your existing renderer or mail view to accept the collected items and load current products as needed. Keep the ordinary `email` channel and its receipt-aware durable adapter. Add other delivery channels to the summary's `via()` and implement their formatters as needed.

### Updating subscription calls

Existing stored subscriptions are retained; do not recreate them to migrate. Use `addSubscription()` for new activation and `updateSubscription()` for settings changes. They are not interchangeable replacements for every legacy `setSubscription()` call: their pending-item behavior differs.

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
