# EmailDigestService

`EmailDigestService@megaphone` manages named subscriptions, collected items, and scheduled email batches. See [Named Email Digests](../guides/email-digests.md) for configuration and consumer integration.

### getDefinition

`getDefinition(digestName)` returns the configured definition after validating its required fields and `maxItems` bound. The result includes the canonical name, schedule, email channel, policy, renderer, and batch size.

**Throws:** `Megaphone.Digests.MissingDefinition` for an unknown or invalid name, or `Megaphone.Digests.InvalidDefinition` for missing configuration or an invalid batch size.

### getSubscription

`getSubscription(notifiable, digestName)` loads the named subscription owned by the Notifiable.

**Return:** A subscription struct, or null when none exists. Fields include `id`, `digestName`, `notifiableType`, `notifiableId`, `mode`, `generation`, `timezone`, `schedule`, `effectiveAt`, and `nextDueAt`. Boundary timestamps use UTC epoch milliseconds. An absent subscription is off.

### setSubscription

Stores a named delivery mode and prospective schedule.

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | | The subscription owner. |
| `digestName` | `string` | true | | A configured named digest. |
| `mode` | `string` | true | | `off`, `immediate`, or `digest`. |
| `timezone` | `string` | false | `UTC` | The confirmed IANA timezone; an empty value also uses UTC. |
| `schedule` | `struct` | false | Definition schedule | `cadence` and `time`, plus `weekday` for weekly schedules. |
| `clock` | `date` | false | `now()` | The effective change time. |

**Return:** The stored subscription struct. Identical choices keep its generation and collected items. A changed mode, timezone, or schedule suppresses obsolete work and begins a new generation. Changing to Immediate does not send previously collected items; Notifications use the ordinary email channel for future immediate email.

### unsubscribe

`unsubscribe(notifiable, digestName, clock=now())` changes only the named subscription to off, keeping its schedule and timezone.

**Return:** The updated subscription struct. The application authorizes the recipient and validates its signed unsubscribe link before calling this method.

### collect

Stores one logical item or its suppression identity without sending mail.

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

**Return:** `{id, status, created}`. `status` describes collection (`collected`, `batched`, or `suppressed`), and `created` says whether a new item was inserted. Repeating the same identity returns the existing item without re-enabling it. Different content or occurrence under that key throws `Megaphone.Digests.IdentityConflict`.

Most Notifications collect through `toEmailDigest()` and `notify()` or `queuePlan()`. Use the direct operation when an application already owns domain publication. Publication replay exposes its original admissions through `NotificationPublication.getCollections()`.

### prepareDue

`prepareDue(limit=100, clock=now())` freezes due subscriptions and persists one durable email delivery for each nonempty batch. The limit accepts integers from 1 to 1000. This operation performs no Provider I/O.

**Return:** An array of batch structs. Fields include `id`, `subscriptionId`, `generation`, `cutoffAt`, `state`, and `deliveryId`. Empty batches have no `deliveryId`. Dispatch nonempty batch delivery IDs after commit through `NotificationDeliveryService` or your configured queue.

### dispatchDue

`dispatchDue(limit=100, clock=now())` also recovers expired digest worker leases and dispatches due digest mail. It leaves unrelated channels alone and uses the existing bounded transport attempts and acceptance tracking.

**Return:** An array of `{deliveryId, result}` structs. `result` is a `DeliveryResult`; inspect `getStatus()` and `getReason()` to distinguish Provider acceptance, suppression, retry, permanent rejection, and ambiguity. The email is dispatched after preparation commits, so invoke this operation outside an application transaction.

### prepareBatch

`prepareBatch(notifiable, batchId)` performs a batch preflight. It checks ownership, subscription generation, consent, and current item eligibility. Newly ineligible items are permanently excluded from this batch.

**Return:** A struct with `status` and `reason`. A ready result also includes `batch`, `subscription`, `definition`, and eligible `items`. This operation can change item eligibility state; use delivery APIs for a read-only history view.

### getCollectionsForEvent

`getCollectionsForEvent(eventId)` returns an array of `{id, status, created:false}` admission records for a saved publication event. Different events can refer to the same collected item.

### Policy and renderer interfaces

`IEmailDigestPolicy` describes the boolean `canCollect`, `canSend`, and `isEligible` checks. `IEmailDigestRenderer.render(notifiable, definition, batch, items)` returns a `BaseNotification` with `toEmail()` content. Implementing the interfaces is optional; the configured components must provide the methods. See [Named Email Digests](../guides/email-digests.md#supplying-consent-and-eligibility) for their responsibilities.

Transport-start guards and collaborator resolution are internal Provider integration methods. Application schedulers use `prepareDue()` or `dispatchDue()`.
