# Delivery operations

Use these APIs after [setting up durable delivery](durable-delivery.md). Your app supplies operator authorization and the delivery filters.

## Browsing delivery history

`DeliveryStore.getPage(maxRows=50, offset=0, constraints=callback)` returns
`{ results, hasMore }` without claiming or sending work. Results are ordered by
updated date descending, then delivery ID. Page sizes are limited to 1–200, with
one extra row fetched to check for another page. Routing data is decoded for you.

The callback receives a query with the `delivery` alias before ordering and
pagination. Use it to add filters, domain joins, and metadata. Check operator
access and add the recipient, namespace, and domain restrictions your app needs.
This method doesn't expose an HTTP endpoint or choose your application's scopes.

## Recovering failed deliveries

`DeliveryStore.requestRecovery(id, requestKey, actorId, expectedVersion,
eligibility, queueAdapter, additionalAttempts=1, actorLabel="")` records who
requested a retry and queues the work in the same transaction. Get
`expectedVersion` from `recoveryVersion(delivery)` and check that the operator
can recover this delivery before calling it.

Your eligibility callback checks current recipient access, preferences, and
domain state, then returns `{ status: "ready" }`. The queue adapter saves its job
in the same datasource transaction. It must not send to a provider there.

You can recover retryable or permanently failed work. Accepted, ambiguous,
suppressed, and processing work can't be recovered. Existing attempt and transport
counts, provider references, and future backoff are preserved. Each request adds
1–5 transports, up to 45 additional transports over the delivery's lifetime.

Repeating the same request key and intent returns the original receipt. Reusing
that key for different intent is a conflict. A stale version or failed eligibility
check creates no receipt or job. Check for external acceptance before requesting
recovery; a permanent state alone doesn't prove the old provider rejected it.

`recoveryHistory(id, maxRows=20, offset=0)` gives you pages of recovery receipts,
including the operator, previous state, counts, evidence, and added allowance.
Resolved-event cleanup removes these receipts with the other delivery diagnostics.
Migration 010002 completes the column addition from 010000. Rolling back 010002
leaves that column for 010000 to remove when rolling back the recovery feature.

Attempt cleanup locks the delivery and checks that it's still terminal before
removing expired diagnostics. Event cleanup locks the event and its deliveries
in ID order, then checks every delivery's terminal state and settlement cutoff.
Recovering work protects the event, attempts, and receipts from cleanup. Settling
it again starts a new retention period. Choose an event cutoff beyond your
supported replay window; the inbox retention setting doesn't define that window.

## Counting delivery states

`DeliveryStore@megaphone.stateCounts(constraints)` returns channel/state/total
groups using distinct delivery IDs. The optional callback gets the same `delivery`
alias as `getPage`. Use it to filter by ownership, namespace, or migration before
counting. You can compare stored states without loading every delivery or writing
queries against Megaphone's tables. These counts tell you the current delivery
state, not whether the recipient received or read the message.

## Limiting event replay

`DurableNotificationService.publish(event, intents, queueAdapter, admitAfter)`
lets you set an occurrence cutoff. Supply the saved domain occurrence time in
`event.createdDate`. A missing or invalid value throws
`Megaphone.Events.OccurrenceRequired`. An occurrence strictly older than the
cutoff returns `{ event: { id: "", created: false, expired: true }, deliveries: [] }`
without saving an event, delivery, or queue job. An occurrence equal to the cutoff
is allowed. Calls without `admitAfter` keep their existing behavior.

You can call `canPublish(occurredAt, admitAfter)` before resolving recipients and
preferences to skip that work for expired events. `publish` checks again before
saving. An expired publication doesn't cancel existing deliveries; their owner
still handles recovery. Imports that don't send and migrations of pending work
can use their own cutoff rules.

Choose an event retention cutoff no later than the admission cutoff, and keep
unresolved work independently. Use the original saved occurrence time on every
producer, not the time a retry runs. If you've pruned event identities, restore
the deduplication evidence before widening the replay window. Admission checks
don't run cleanup or remove queue and provider records for you.

`pruneResolvedEvents(eventsBefore, deliveriesBefore, limit=100, constraints)`
accepts a backend query callback. It filters candidates before the batch limit
and runs again against the locked event before deletion. Use it to limit cleanup
to your namespace or keep events with external queue or import references.
Megaphone still checks terminal states, settlement cutoffs, and parent/child
locks. Your external reference updates need compatible locking so they don't
race event deletion.

`DeliveryStore.find(id, lockRow=false)` can lock a delivery for lifecycle work in
your app. Set `lockRow=true` inside your transaction and keep that transaction
open until the external reference update is finished. Recovery, claims,
completion, and retention use the same delivery lock. The lookup doesn't start
a transaction or check caller authorization for you.

You can also pass `beforePrune(event, deliveries)` to `pruneResolvedEvents`.
It runs inside the cleanup transaction after the event and terminal deliveries
are locked and checked, but before they're removed. The callback gets copies of
the stored rows. Use it to save a small retirement receipt in the same datasource.
If it throws, deletion rolls back. Keep external I/O out of the callback. Pending,
ambiguous, or otherwise ineligible events never reach it. Your app still manages
its references and any evidence it keeps.
