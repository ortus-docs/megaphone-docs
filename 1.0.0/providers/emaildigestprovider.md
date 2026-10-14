# EmailDigestProvider

Want to send a weekly summary instead of an email for every Notification? The `EmailDigestProvider` collects content for a named digest. In scheduled mode the module scheduler sends them later; in immediate mode each new item becomes a one-item batch through the same renderer and [EmailProvider](emailprovider.md) channel.

Each named digest has its own subscription and schedule for each Notifiable. A user can subscribe to product updates and organization news independently, or unsubscribe from one while continuing to receive the other.

### Requirements

Apply Megaphone's digest migration before collecting Notifications. The collection tables, event store, and delivery store must use the same transactional datasource. Your application supplies the digest's consent and eligibility policy, renderer, and recipient resolver.

Sending a batch requires an email channel with a receipt-aware durable adapter. Scheduled collection does not send mail. For immediate subscriptions, `notify()` dispatches a one-item batch synchronously and `queuePlan()` creates a queued durable delivery. See [Named Email Digests](../guides/email-digests.md) for the complete setup.

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
        "itemKey": variables.digests.itemKey( [ "product.updated", variables.updateEventId, 1 ] ),
        "occurredAt": variables.digests.occurredAt( variables.updateOccurredAt ),
        "data": { "productId": variables.productId }
    };
}
```

`itemKey` is a stable event identity, up to 190 characters. Repeated requests and overlapping matching passes reuse it. `occurredAt` is the persisted occurrence time in UTC epoch milliseconds; use the original occurrence rather than the time a retry runs. `data` contains JSON-safe values your renderer and eligibility policy need.

The Provider calls `toEmailDigest()` through the usual `routeForType()` helper. Use `notify()` for synchronous collection, or `plan()` and `queuePlan()` for captured transactional publication. Admission is separate from email acceptance; inspect the resulting durable delivery for the Provider outcome. A `NotificationPublication` exposes collected item admissions through `getCollections()`, separately from `getDeliveries()`.

Keep `via()` returning `[ "emailDigest" ]` for either timing. Do not add `email` to the same Notification to support Immediate: that creates a second independent delivery path and can send a duplicate email. The definition's `emailChannel` already supplies the underlying email transport. Megaphone reads Off, Immediate, or Digest from the named subscription. Immediate uses the digest renderer with one item; no separate source `toEmail()` or ordinary email route is needed. An immediate-only definition can omit `schedule`. Changing a subscription does not send or replay earlier items. See [Choosing a Delivery Mode](../guides/email-digests.md#choosing-a-delivery-mode) for an example.

See [An Immediate-only Digest Policy](../guides/email-digests.md#an-immediate-only-digest-policy) for a complete On/Off product example.

See [Why Use an Immediate Digest?](../guides/email-digests.md#why-use-an-immediate-digest) for a catalog example comparing ordinary immediate email, an immediate digest, and both products.

See [Use One Email Route per Product-update Notification](../guides/email-digests.md#use-one-email-route-per-product-update-notification) for the routing and subscription-mode comparison.
