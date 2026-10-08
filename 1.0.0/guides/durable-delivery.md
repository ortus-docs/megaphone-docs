# Durable delivery

Existing synchronous consumers can keep using their current APIs. To use durable
delivery, apply the delivery and attempt migrations, including the delivery
timestamp precision migration. It keeps MySQL from rounding newly queued work
into the next second. PostgreSQL already preserves that precision.
`DurableNotificationService@megaphone.publish(event, intents, queueAdapter)`
records an event and its recipient/channel/device work in one transaction.
Each intent supplies `recipientType`, `recipientId`, `channel`, optional
`deviceId`, explicit `enabled`, `routingData`, `routingHash`, and optional
`availableDate`. Disabled channels are saved as suppressed. Enabling one later
doesn't revive old work. Publishing the same event again returns the existing
work without adding recipients or channels that became eligible afterward.

Your optional queue adapter implements `enqueue(delivery)`. Save a job reference
in the same datasource and transaction, without sending to a provider. A queue
insertion failure rolls back publication, even if the caller catches it inside
the domain transaction. Without an adapter, use `DeliveryStore.due(limit, clock)`
from a scheduler. Duplicate jobs are safe because only the current claim token
can start transport.

`due(limit=100, clock=now(), constraints)` takes an optional backend query callback
before ordering and limiting the batch. Use it to exclude deliveries that already
have live queue jobs so they don't crowd out missing jobs. Keep queue-specific
queries in your adapter and recheck under its concurrency guard before enqueueing.
Selecting candidates alone doesn't prevent two schedulers from acting on them.

`DeliveryStore@megaphone` provides `enqueue`, `find`, `forEvent`, `claim`,
`startTransport`, `complete`, `due`, `attempts`, and `recoverExpired`.
Constructor properties let you set `table`, `attemptsTable`, `eventsTable`, and
`queryOptions`. Use the domain transaction's datasource for events, deliveries,
and queue jobs. Queue delivery IDs rather than serialized users or credentials.

`DeliveryDispatcher@megaphone.dispatch(id, eligibility, sender, leaseSeconds=60,
maxAttempts=5, retrySeconds=30)` claims work and calls your callbacks:

- `eligibility(delivery)` prepares content and checks current access, preferences,
  registration ownership, feature availability, and source revision just before
  transport. Return `status` as `ready`, `suppressed`, `deferred`, or `permanent`,
  with optional safe reason, retry date, and prepared content. You supply the
  application's authorization rules.
- `sender(delivery, prepared)` sends through your channel adapter. Return `outcome`
  as `accepted`, `retryable`, `permanent`, or `ambiguous`, with optional provider
  reference, safe reason, and retry date. Use `retryable` only when you know the
  provider rejected the request before acceptance. An uncertain timeout is
  `ambiguous`.

The transport-start marker must succeed before sending. If a lease expires before
that point, a new worker can retry and the old token can no longer start transport.
If it expires or throws afterward, acceptance is ambiguous and automatic resends
stop. A late result from that same lease can still record acceptance. Establish a
definite outcome before retrying. Provider acceptance doesn't mean the recipient
received or read the message.

Feature pauses and preparation failures before transport don't use up the actual
send allowance. `attemptCount` counts claims; `transportCount` counts starts.
`recoverExpired(limit=100, clock)` checks expired processing rows in batches.
Run it alongside your due-work scheduler so lost jobs don't leave work stranded.
Claim and policy state is committed before calling the provider. Don't wrap
network I/O in a new domain transaction in your sender callback.

`reconcileAccepted(id, acceptance, clock=now())` settles work using a verified
receipt, including late acceptance or an import from the previous delivery owner.
The callback receives the immutable delivery and returns `{ accepted: false }`
or `{ accepted: true, providerReference: "receipt-id" }`. Verify that the receipt
belongs to that delivery. Keep the callback local, without external I/O.
Confirmed acceptance prevents queued or running work from sending again.
Existing attempt outcomes stay attached to their original attempts. Authorize
this operation in your app before exposing it through an endpoint.

`pruneAttempts(beforeDate, limit=200)` removes diagnostics for terminal deliveries
and keeps the provider reference. `pruneResolvedEvents(eventsBefore,
deliveriesBefore, limit=100)` removes event identities and their work only after
the replay window, when every delivery is terminal and old enough. Pending or
ambiguous work keeps its evidence. Reject old events outside your replay window
before publishing; use `importNotification` when you only need to import history.
Neither cleanup method removes inbox entries or domain audit history.

Set your cutoffs in scheduled configuration. For example, you might keep terminal
attempt diagnostics for 90 days and resolved event identities for 365 days.
The event cutoff must be beyond your supported replay window. The inbox count
cap is a separate setting.

See [delivery operations](delivery-operations.md) for recovery, state counts, and replay cutoffs.
