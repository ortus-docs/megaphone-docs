# EmailDigestProvider

Want to send a weekly summary instead of an email for every Notification? The `EmailDigestProvider` collects content for a named digest. Your scheduler sends the collected items later through an [EmailProvider](emailprovider.md) channel.

Each named digest has its own subscription and schedule for each Notifiable. A user can subscribe to product updates and organization news independently, or unsubscribe from one while continuing to receive the other.

### Requirements

Apply Megaphone's digest migration before collecting Notifications. The collection tables, event store, and delivery store must use the same transactional datasource. Your application supplies the digest's consent and eligibility policy, renderer, and recipient resolver.

Sending a batch requires an email channel with a receipt-aware durable adapter. The collection Provider does not send mail or require a cbq worker. See [Named Email Digests](../guides/email-digests.md) for the complete setup.

### Configuration

Register a collection channel as you would any other Provider:

```cfml
"emailDigest": {
    "provider": "EmailDigestProvider@megaphone",
    "properties": {}
}
```

Define the named digests under `moduleSettings.megaphone.digests`. The schedule belongs to a definition and can be overridden for an individual subscription:

```cfml
"digests": {
    "productUpdates": {
        "schedule": "everyWeekOn 1 09:00",
        "emailChannel": "email",
        "policy": "ProductDigestPolicy",
        "renderer": "ProductDigestRenderer"
    }
}
```

`emailDigestBatch` is Megaphone's internal mail channel. It is registered automatically when named digests are configured. Application Notifications select the collection channel; they do not select the internal batch channel.

### toEmailDigest

Return a struct identifying the digest and the logical item to collect:

```cfml
public struct function toEmailDigest( required any notifiable ) {
    return {
        "digestName": "productUpdates",
        "itemKey": variables.updateEventId,
        "occurredAt": variables.updateOccurredAt,
        "data": { "productId": variables.productId }
    };
}
```

`itemKey` is a stable event identity, up to 190 characters. Repeated requests and overlapping matching passes reuse it. `occurredAt` is the persisted occurrence time in UTC epoch milliseconds; use the original occurrence rather than the time a retry runs. `data` contains JSON-safe values your renderer and eligibility policy need.

The Provider calls `toEmailDigest()` through the usual `routeForType()` helper. Use `notify()` for synchronous collection, or `plan()` and `queuePlan()` for captured transactional publication. Both collect without recording an email acceptance. A `NotificationPublication` exposes collected item admissions through `getCollections()`, separately from `getDeliveries()`.

To choose Off, Immediate email, or Digest, have the Notification's `via()` method read its named subscription. Immediate email uses the ordinary email channel and `toEmail()`; changing a subscription does not send or replay earlier items. See [Choosing a Delivery Mode](../guides/email-digests.md#choosing-a-delivery-mode) for an example.
