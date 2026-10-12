# Delivery Operations

A queued notification can have several deliveries: an inbox entry, an email, and a push for each device. `NotificationDeliveryService` lets you inspect those deliveries, reconcile Provider receipts, and recover work that can safely be tried again.

### Looking up deliveries

Inject `NotificationDeliveryService@megaphone`. You can fetch a saved event or delivery by ID, or list the deliveries for an event:

```js
var event = notificationDeliveryService.getEvent( eventId );
var deliveries = notificationDeliveryService.getDeliveriesForEvent( eventId );
var delivery = notificationDeliveryService.getDelivery( deliveryId );
```

For an operator screen, `getDeliveries()` returns a page with `getResults()`, `getHasMore()`, and `getMemento()`. `getStateCounts()` provides counts, `getAttempts(id)` shows transport attempts, and `getRecoveryHistory(id)` returns a `NotificationRecoveryPage` with the same page accessors.

Events, deliveries, attempts, and recovery receipts expose their values through accessors and `getMemento()`. A delivery's `getAttributes()` contains application-selected display columns separately from its identity. Check operator permissions in your application before exposing any of this history.

### Requesting recovery

Recovery gives a retryable or permanently rejected delivery another transport allowance. First, use `getRecoveryVersion(delivery)` to capture the version the operator is acting on. Pass that value as `expectedVersion` when calling `requestRecovery()`:

| Argument | Purpose |
| --- | --- |
| `id` | The delivery to recover. |
| `requestKey` | A stable key for this recovery request. |
| `actorId` | The operator requesting recovery. |
| `expectedVersion` | The token from `getRecoveryVersion()`. |
| `eligibility` | A callback receiving the current delivery and returning a `DeliveryPreparation`. |
| `queueAdapter` | An adapter that enqueues the recovered delivery in the same transaction. |
| `additionalAttempts` | The added transport allowance; defaults to `1`. |
| `actorLabel` | An optional display label for the operator. |

Megaphone checks the current version and eligibility, records a recovery receipt, and inserts the job atomically. It returns a `DeliveryRecoveryResult`. Repeating the same request key returns the original receipt without inserting another job.

Only retryable or permanently rejected work can recover. Accepted, ambiguous, suppressed, and processing deliveries can't be recovered this way. The default limits are five additional transports per request and 45 over a delivery's lifetime. Change `maxRecoveryAttempts` and `maxRecoveryAllowance` in the module's `durable` settings if your application needs different limits.

### Reconciling acceptance

Sometimes a Provider receipt arrives after Megaphone recorded an uncertain outcome. Use `reconcileAccepted(id, acceptance)` to record that evidence without sending again.

The `acceptance` callback receives the current delivery. Your application validates the local receipt and returns a struct with `accepted` and `providerReference`. This can also be authoritative evidence imported from an older delivery system. Reconciliation updates acceptance while preserving the older attempt diagnostics.

Keep Provider calls outside this callback: it runs inside the reconciliation transaction.

### Scheduled maintenance

`recoverExpired()` handles expired worker leases. For retention, use `pruneAttempts(beforeDate, limit)` and `pruneResolvedEvents(eventsBefore, deliveriesBefore, limit)` to work in bounded batches.

If your application needs to save retirement evidence before an event is removed, pass a `beforePrune(event, deliveries)` callback. It receives objects after owner locks and terminal-state checks, within the pruning transaction. Save the local evidence there and keep external I/O outside the callback.

For schema changes and existing saved work, see the [Upgrade Guide](../upgrade-guide.md).
