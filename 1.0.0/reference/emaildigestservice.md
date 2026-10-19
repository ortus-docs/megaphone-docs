# EmailDigestService

Megaphone 1.3.0 also exposes [DigestService](digestservice.md) for provider-independent summaries and subscription lifecycle methods. The legacy methods below retain their existing pending-item behavior.

`EmailDigestService@megaphone` manages named subscriptions, collected items, and scheduled email batches. See [Named Email Digests](../guides/email-digests.md) for configuration and consumer integration.

### getDefinition

`getDefinition(digestName)` returns the configured definition after validating its required fields. The result includes the canonical name, schedule, email channel, policy, renderer, and optional selection callback.

**Throws:** `Megaphone.Digests.MissingDefinition` for an unknown or invalid name, or `Megaphone.Digests.InvalidDefinition` for missing configuration.

### getSubscription

`getSubscription(notifiable, digestName)` loads the named subscription owned by the Notifiable.

**Return:** A subscription struct, or null when none exists. Fields include `id`, `digestName`, `notifiableType`, `notifiableId`, `mode`, `generation`, `timezone`, `schedule`, `effectiveAt`, and `lastCutoffAt`. Boundary timestamps use UTC epoch milliseconds. An absent subscription is off.

### setSubscription

Stores a named delivery mode and prospective schedule.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The subscription owner. |
| `digestName` | `string` | true | | A configured named digest. |
| `mode` | `string` | true | | `off`, `immediate`, or `digest`. |
| `timezone` | `string` | false | Definition/module scheduler | The confirmed Java timezone identifier, validated by the native scheduler. |
| `schedule` | `string`, `struct`, or unstarted task instance | false | Named definition reference | A native frequency shortcut or `{cadence, arguments}` override. Executable closures belong in the named definition; a callback-free task instance can be persisted as an override. |
| `clock` | `date` | false | `now()` | The effective change time. |

**Return:** The stored subscription struct. Identical choices keep its generation and collected items. A changed mode, timezone, or schedule suppresses obsolete work and begins a new generation. Changing to Immediate does not send previously collected items. Future items published through `emailDigest` become one-item batches using the named digest renderer. A definition without `schedule` supports immediate subscriptions; scheduled mode requires a configured schedule or explicit override.

### unsubscribe

`unsubscribe(notifiable, digestName, clock=now())` changes only the named subscription to off, keeping its schedule and timezone.

**Return:** The updated subscription struct. The application authorizes the recipient and validates its signed unsubscribe link before calling this method.

### collect

Stores one logical item or its suppression identity without Provider I/O. Immediate mode also freezes that new item and creates its durable batch delivery in the same transaction.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The collection owner. |
| `digestName` | `string` | true | | A configured digest name. |
| `itemKey` | `string` | true | | A stable logical item identity, up to 190 characters. |
| `data` | `struct` | true | | JSON-safe content references for eligibility and rendering. |
| `occurredAt` | `numeric` | true | | The persisted occurrence time in UTC epoch milliseconds; no later than collection. |
| `enabled` | `boolean` | false | `true` | The captured channel decision. False remains suppressed. |
| `eventId` | `string` | false | `""` | The saved publication event, used by `queuePlan()` to retain admission links. |
| `clock` | `date` | false | `now()` | Admission time; controls cutoff membership. |

**Return:** `{id, status, created}`, plus `batchId` and `deliveryId` for a nonempty immediate batch. `status` describes collection (`collected`, `batched`, `discarded`, or `suppressed`), and `created` says whether a new item was inserted. Repeating the same identity returns the existing item without re-enabling it. Different content or occurrence under that key throws `Megaphone.Digests.IdentityConflict`.

Most Notifications collect through `toEmailDigest()` and `notify()` or `queuePlan()`. Use the direct operation when an application already owns domain publication. Publication replay exposes its original admissions through `NotificationPublication.getCollections()`.

### prepareDigest

`prepareDigest(digestName, cutoff, timezone, schedule, limit=100)` freezes candidates for one scheduled boundary and persists one durable email delivery per nonempty batch. `cutoff` is integer UTC epoch milliseconds; omitted timezone/schedule use named defaults. The recipient limit accepts integers from 1 to 1000. This operation performs no Provider I/O. The native module scheduler calls it automatically and drains the group in bounded transactions.

**Return:** An array of batch structs with `id`, `subscriptionId`, `generation`, `cutoffAt`, `state`, and `deliveryId`. Empty batches have no delivery ID. Dispatch after commit. Unselected pre-cutoff items are discarded permanently. A repeated cutoff creates no additional batch. Digest/event/delivery stores must share the same datasource or `Megaphone.Digests.CrossDatasource` is thrown.

### registerDigest

`registerDigest(digestName, definition)` adds a definition and registers its native task on the existing `cbScheduler@megaphone`, starting it when that scheduler is already running. Definitions with a schedule accept shortcut strings, `{cadence, arguments}`, configuration closures, or unstarted native task templates. Immediate-only definitions omit the schedule and create no calendar task. Duplicate names throw `Megaphone.Digests.DuplicateDefinition`.

**Return:** The validated definition. Runtime definitions remain application configuration; register them on every node and again on restart.

### itemKey and occurredAt

`itemKey(parts)` returns a 64-character SHA-256 identity from an ordered array of stable JSON-safe domain identity parts. `occurredAt(occurrence)` converts a saved date to integer UTC epoch milliseconds.

```cfml
"itemKey": digests.itemKey( [ "product.updated", savedEvent.getId(), 1 ] ),
"occurredAt": digests.occurredAt( savedEvent.getOccurredDate() )
```

Reuse the same identity and original event date on retry.

### dispatchDue

`dispatchDue(limit=100, clock=now())` recovers expired digest worker leases and dispatches pending digest mail. It does not create batches. It leaves unrelated channels alone and uses the existing bounded transport attempts and acceptance tracking.

**Return:** An array of `{deliveryId, result}` structs. `result` is a `DeliveryResult`; inspect `getStatus()` and `getReason()` to distinguish Provider acceptance, suppression, retry, permanent rejection, and ambiguity. The email is dispatched after preparation commits, so invoke this operation outside an application transaction.

### prepareBatch

`prepareBatch(notifiable, batchId)` performs a batch preflight. It checks ownership, subscription generation, consent, and current item eligibility. Newly ineligible items are permanently excluded from this batch.

**Return:** A struct with `status` and `reason`. A ready result also includes `batch`, `subscription`, `definition`, and eligible `items`. This operation can change item eligibility state; use delivery APIs for a read-only history view.

### getCollectionsForEvent

`getCollectionsForEvent(eventId)` returns an array of `{id, status, created:false}` admission records for a saved publication event. Different events can refer to the same collected item.

### Policy and renderer interfaces

`IEmailDigestPolicy` describes the boolean `canCollect`, `canSend`, and `isEligible` checks. `IEmailDigestRenderer.render(notifiable, definition, batch, items)` returns a `BaseNotification` with `toEmail()` content. Implementing the interfaces is optional; the configured components must provide the methods. See [Named Email Digests](../guides/email-digests.md#supplying-consent-and-eligibility) for their responsibilities.

Transport-start guards and collaborator resolution are internal Provider integration methods. The module scheduler owns normal preparation and dispatch. `useScheduler()` and `refreshSchedules()` support module lifecycle integration.
