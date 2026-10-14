# Named Email Digests

A named digest lets a Notifiable receive a summary on its own schedule. Notifications collect through [EmailDigestProvider](../providers/emaildigestprovider.md). A scheduled call freezes the due items and sends one email through your configured email channel.

Each digest has its own subscription, calendar schedule, and timezone. The application decides what the digest contains and whether each item is still eligible when the email is prepared.

### Configuring a digest

Apply `2026_10_14_120000_create_megaphone_email_digests` through your application's migration ledger. It adds `megaphone_digest_subscriptions`, `megaphone_digest_items`, `megaphone_digest_batches`, and `megaphone_digest_admissions`. Use the same transactional datasource for these tables, notification events, and deliveries.

Configure the collection channel, named definition, and storage:

```cfml
moduleSettings.megaphone = {
    "channels": {
        "email": {
            "provider": "EmailProvider@megaphone",
            "properties": { "durableAdapter": "ReceiptAwareEmailAdapter" }
        },
        "emailDigest": { "provider": "EmailDigestProvider@megaphone", "properties": {} }
    },
    "digests": {
        "productUpdates": {
            "schedule": { "cadence": "weekly", "weekday": 1, "time": "09:00" },
            "emailChannel": "email",
            "policy": "ProductDigestPolicy",
            "renderer": "ProductDigestRenderer",
            "maxItems": 500
        }
    },
    "storage": {
        "digests": { "datasource": "applicationDatabase" },
        "events": { "datasource": "applicationDatabase" },
        "deliveries": { "datasource": "applicationDatabase" }
    },
    "queue": { "notifiableResolver": "NotifiableResolver" }
};
```

`NotifiableResolver.resolve(type, id)` loads the current Notifiable or returns null when it no longer exists. Use your existing resolver if the application already uses [Durable Delivery](durable-delivery.md).

The named definition provides the default schedule. `maxItems` bounds one batch to 1–1000 items and defaults to 500. Overflow remains collected for the next scheduled batch. `storage.digests` accepts `datasource`, `queryOptions`, `table` for subscriptions, and `itemsTable`, `batchesTable`, and `admissionsTable` for the other tables. Custom names need matching application migrations.

### Choosing a delivery mode

Subscriptions start off. Store the user's choice using `EmailDigestService@megaphone`:

```cfml
var digests = wirebox.getInstance( "EmailDigestService@megaphone" );

digests.setSubscription(
    notifiable = user,
    digestName = "productUpdates",
    mode = "digest",
    timezone = user.getConfirmedTimezone() ?: "UTC"
);
```

Use `off`, `immediate`, or `digest`. The application supplies a confirmed IANA timezone; a missing zone falls back to UTC and an invalid supplied zone is rejected. Pass a `schedule` to give this subscription a different cadence or time.

Have the Notification choose its channel from that named subscription:

```cfml
component extends="megaphone.models.BaseNotification" accessors="true" {
    property name="digests" inject="EmailDigestService@megaphone";
    property name="productId";
    property name="updateEventId";
    property name="updateOccurredAt";

    public array function via( required any notifiable ) {
        var subscription = variables.digests.getSubscription( arguments.notifiable, "productUpdates" );
        if ( isNull( subscription ) || subscription.mode == "off" ) {
            return [];
        }
        return [ subscription.mode == "digest" ? "emailDigest" : "email" ];
    }

    public struct function toEmailDigest( required any notifiable ) {
        return {
            "digestName": "productUpdates",
            "itemKey": variables.updateEventId,
            "occurredAt": variables.updateOccurredAt,
            "data": { "productId": variables.productId }
        };
    }

    public any function toEmail( required any notifiable, required any newMail ) {
        return arguments.newMail(
            to = arguments.notifiable.routeNotificationForEmail(),
            from = "updates@example.com",
            subject = "A product was updated",
            type = "text",
            body = "Product " & variables.productId & " has an update."
        );
    }
}
```

Call `notify()`, `queue()`, or `queuePlan()` as usual. `plan()` only captures the collection intent; publication stores the collection in the caller's transaction. The default cbq adapter is not invoked for collection-only work, since no mail delivery job exists yet.

A repeated identical subscription update keeps pending items. Changing mode, timezone, or schedule starts a new generation and suppresses obsolete collected work. Returning to Digest does not replay it. Call `setSubscription()` transactionally when the application changes that preference or confirmed timezone. Changes to a definition's default do not rewrite existing subscription schedules.

### Scheduling in local time

A schedule supplies `cadence` (`daily` or `weekly`) and `time` (`HH:mm`). Weekly schedules also supply `weekday`, using Monday `1` through Sunday `7`.

Calendar scheduling keeps the configured local hour through daylight saving changes. A nonexistent local time shifts forward by the gap; a repeated local time uses the earlier offset and runs once. The persisted boundaries are UTC epoch milliseconds, independent of the server timezone.

For example, an Audition Discovery consumer can configure Monday at 9 a.m. and pass the account's confirmed timezone with UTC fallback. That schedule and the discovery rules belong to the application. Other digests can use different definitions and individual schedules.

### Supplying consent and eligibility

The definition's `policy` is a WireBox mapping or component implementing `IEmailDigestPolicy`, or the same methods:

| Method | What it checks |
| --- | --- |
| `canCollect(notifiable, definition, item)` | Whether this item may enter collection under the current preference and consent policy. |
| `canSend(notifiable, definition, subscription)` | Whether the recipient currently permits this named digest. |
| `isEligible(notifiable, definition, item)` | Whether the item is still visible and eligible for this recipient. |

Each method returns a boolean and performs local reads. Check authoritative preference and domain state rather than cached Notifiable fields or saved item content. Policy errors prevent a send. A global `queue.eligibilityPolicy` also runs for the internal `emailDigestBatch` event; make it recognize the batch reference and leave item decisions to the named policy.

Off, disabled, or denied items become suppression records. Their identities are not re-enabled by a later retry. Occurrences before the subscription's effective boundary are also suppressed. A changed preference cannot cause an old item to be collected again.

### Rendering the summary

The definition's `renderer` implements `IEmailDigestRenderer`, or its `render(notifiable, definition, batch, items)` method. Return a `BaseNotification` that implements `toEmail()`:

```cfml
component {
    public BaseNotification function render(
        required any notifiable,
        required struct definition,
        required struct batch,
        required array items
    ) {
        return newDigestEmail().populate( {
            "digestName": arguments.definition.name,
            "items": arguments.items
        } );
    }

    private any function newDigestEmail() provider="ProductDigestEmail" {
    }
}
```

`ProductDigestEmail` is your application's Notification. It loads current product content from each item's `data`, formats the summary in `toEmail()`, and includes a signed unsubscribe link for this digest. Megaphone assigns the stable batch ID to the rendered Notification and prepares it through the configured receipt-aware email channel.

Consent, subscription generation, and item eligibility are checked again after rendering, immediately before transport starts. If eligibility changed during rendering, the prepared message is discarded and rebuilt on retry. An item omitted from a batch never rejoins that batch. An empty eligible batch is suppressed without sending. Once transport starts, a preference change cannot retract that Provider request.

### Dispatching due batches

Call from your scheduler after application transactions commit:

```cfml
wirebox.getInstance( "EmailDigestService@megaphone" ).dispatchDue( limit = 100 );
```

`dispatchDue()` freezes due subscriptions, recovers expired digest worker leases, and dispatches due digest mail. It does not recover or send unrelated notification channels. It needs no cbq worker or automatic scheduler registration.

If your application already handles delivery jobs, call `prepareDue()` instead. It freezes membership and returns batch structs with a `deliveryId` for each nonempty batch. Dispatch those IDs through `NotificationDeliveryService` or your existing queue adapter after commit. An empty batch advances its schedule without creating a mail delivery.

The subscription row lock makes overlapping scheduler calls safe. Items admitted strictly before a cutoff enter that batch; items admitted at or after it go to the next. Backdating an occurrence cannot change closed membership. Batch IDs, cutoffs, and original item membership stay stable during retries. Missed periods advance one calendar period per scheduler call, allowing bounded catch-up.

### Inspecting collection and email

`NotificationPublication.getCollections()` returns `{id, status, created}` admission results. An item can be `collected`, `batched`, or `suppressed`; these states do not mean an email was sent. Event admission links let overlapping events refer to one item and let either event replay return its original collection identity.

A nonempty batch has a separate durable email delivery. Inspect its Provider outcome and attempts using `NotificationDeliveryService.getDelivery(batch.deliveryId)` and `getAttempts(batch.deliveryId)`. Batch state describes frozen membership; email acceptance is recorded on the delivery.

Confirmed retryable failures use the same batch and delivery identity. Unknown outcomes remain `ambiguous` and do not automatically resend. See [Delivery Operations](delivery-operations.md) for acceptance receipts, authoritative rejection reconciliation, and bounded operator recovery. A plain cbMailServices success callback is insufficient for durable acceptance.

### Unsubscribing from one digest

After validating an application-signed link bound to the recipient and digest name, call:

```cfml
digests.unsubscribe( notifiable = user, digestName = "productUpdates" );
```

This turns off only `productUpdates` and suppresses its obsolete collection. It does not change other digest subscriptions, general channel preferences, push registrations, or transactional email settings. The application supplies the handler, signature validation, and authorization.

Keep stable item identities when pruning old content. Deleting collection identities would allow duplicate admission and historical replay. See [EmailDigestService](../reference/emaildigestservice.md) for the public operations and return values.
