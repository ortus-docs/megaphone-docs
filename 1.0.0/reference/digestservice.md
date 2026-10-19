# DigestService

`DigestService@megaphone` manages named digest subscriptions and publishes summaries through ordinary Notifications. These methods require Megaphone 1.3.0. See [Digests](../guides/digests.md) for the CommuniArts example.

### addSubscription

Creates or reactivates a named subscription.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The subscription owner. |
| `digestName` | `string` | true | | A configured digest name. |
| `mode` | `string` | false | `digest` | `digest` for scheduled summaries or `immediate` for one-item summaries. |
| `timezone` | `string` | false | Definition/module scheduler | A timezone accepted by the ColdBox scheduler. |
| `schedule` | Shortcut, struct, or unstarted task | false | Named definition | A serializable schedule override. Executable closures belong in the named definition. |
| `clock` | `date` | false | `now()` | When the change takes effect. |

**Return:** The stored subscription. An identical active subscription is unchanged. Conflicting active settings raise `Megaphone.Digests.SubscriptionConflict`; use `updateSubscription()` instead.

A missing subscription remains disabled until this method or a transfer activates it. Suppressed identities do not replay after activation.

### getSubscription

`getSubscription(notifiable, digestName)` reads the stored subscription.

**Return:** A subscription struct, or null when missing. Fields include `id`, `digestName`, `notifiableType`, `notifiableId`, `mode`, `resumeMode`, `generation`, `timezone`, `schedule`, `effectiveAt`, and `lastCutoffAt`. Boundary timestamps use UTC epoch milliseconds. `mode="off"` is stopped; `resumeMode` retains its saved timing mode.

Use this state to display digest preferences. Delivery channel preferences remain separate.

### updateSubscription

`updateSubscription(notifiable, digestName, mode?, timezone?, schedule?, clock=now())` changes saved settings while preserving eligible pending items. Omitted settings keep their current values.

**Return:** The stored subscription. An identical update is unchanged. Updating a missing subscription raises `Megaphone.Digests.SubscriptionMissing`. A stopped subscription stays stopped.

A timing change invalidates obsolete unstarted batches and computes a future cutoff. Items from batches whose transport has started are not regrouped. To switch between scheduled and immediate modes, stop the subscription, update its saved mode, then resume.

### stopSubscription

`stopSubscription(notifiable, digestName, clock=now())` disables collection and suppresses deliveries whose transport has not started.

**Return:** The stored subscription, or null when missing. Repeated calls do nothing. In-flight and uncertain outcomes remain available for reconciliation.

### resumeSubscription

`resumeSubscription(notifiable, digestName, clock=now())` activates an existing subscription with its saved settings.

**Return:** The stored subscription. An active subscription is unchanged. A missing subscription raises `Megaphone.Digests.SubscriptionMissing`. Only future items are admitted; previously suppressed or discarded items do not replay.

### transferSubscription

`transferSubscription(notifiable, fromDigest, toDigest, pendingItems, clock=now())` stops one named subscription and activates another for the same Notifiable in one transaction.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The shared subscription owner. |
| `fromDigest` | `string` | true | | The existing source subscription. |
| `toDigest` | `string` | true | | A different configured digest. |
| `pendingItems` | `string` | true | | `transfer` or `discard`; there is no default. |
| `clock` | `date` | false | `now()` | When the transfer takes effect. |

`transfer` moves eligible unsent items and keeps their source admission history. The destination policy must approve through `canTransfer(notifiable, sourceDefinition, destinationDefinition, items)`. Missing approval, conflicting content, or a policy error aborts the whole transaction. Destination collection and eligibility checks still apply. `discard` suppresses source pending items without requiring format approval.

An existing destination keeps its settings and pending items. A stopped destination resumes. A missing destination uses its definition's defaults. Repeating a transfer from a stopped source to an active destination is unchanged.

**Return:** A struct with `source`, `destination`, and counts named `transferred`, `deduplicated`, `suppressed`, and `nontransferable`.

Once any child delivery starts, its entire batch is nontransferable. Remaining unstarted source deliveries are suppressed; in-flight outcomes remain auditable. Transfers require the shared transactional datasource used by events, deliveries, and digest storage.

**Throws:** `Megaphone.Digests.InvalidTransfer` for identical names or an invalid disposition, and `Megaphone.Digests.SubscriptionMissing` for a missing source. See [Moving from Daily to Weekly](../guides/digests.md#moving-from-daily-to-weekly) for transfer approval and duplicate-item behavior.

### getDefinition and registerDigest

`getDefinition(digestName)` validates a named definition and returns its canonical name and settings. A generic definition supplies `policy`, `notification`, optional `selectItems`, and a schedule unless it is immediate-only. It contains neither `emailChannel` nor `renderer`.

`registerDigest(digestName, definition)` adds a definition to the existing module scheduler, starting its task when the scheduler is already running. Definitions accept native shortcut strings, schedule closures, and unstarted task templates. Register runtime definitions on every application node and again on restart. Duplicate names raise `Megaphone.Digests.DuplicateDefinition`.

### itemKey and occurredAt

`itemKey(parts)` generates a stable SHA-256 key from ordered JSON-safe identity parts. `occurredAt(occurrence)` converts a saved date to integer UTC epoch milliseconds.

```cfml
"itemKey": digests.itemKey( [ "audition.published", savedEvent.getId(), 1 ] ),
"occurredAt": digests.occurredAt( savedEvent.getOccurredDate() )
```

Reuse the saved event ID and original occurrence date on retry.

### Collection and publication

`collect()` and `getCollectionsForEvent()` retain the [email digest service's item and admission contracts](emaildigestservice.md#collect). Generic Providers call `toDigest()` and supply the digest name from channel configuration.

`prepareDigest(digestName, cutoff, timezone?, schedule?, limit=100)` freezes eligible items at a UTC epoch-millisecond cutoff and saves one publication job per nonempty batch. `dispatchDue(limit=100, clock=now())` processes pending digest jobs and linked summary deliveries. The native module scheduler calls these methods automatically.

For a generic definition, publication creates the configured summary Notification with `batchId`, `digestName`, `cutoffAt`, and selected `items`. Its `via()` chooses ordinary delivery channels once. Each channel has an independent durable delivery. Repeated publication does not add channels or create another summary event. Empty routing is saved as a suppressed publication. Summary Notifications cannot route to collection channels.

A publication job or collection admission does not prove external delivery. Inspect each summary delivery's status and attempts for Provider outcomes.

### Compatibility with EmailDigestService

The module resolves `DigestService@megaphone` and `EmailDigestService@megaphone` to the same service. Existing email definitions, `toEmailDigest()`, renderer contracts, `emailChannel`, `setSubscription()`, and `unsubscribe()` remain supported.

Legacy `setSubscription()` keeps its existing pending-item invalidation behavior. The new `updateSubscription()` preserves eligible pending items. See the [Upgrade Guide](../upgrade-guide.md) before changing APIs in an existing application.
