# Event identities

Apply the `megaphone_events` migration to use `EventStore@megaphone`.
`record(namespace, eventKey, version, type, payload, payloadHash, createdDate)`
stores an immutable payload snapshot. Repeating the call returns the existing
event. Supply a stable hash of your canonical domain payload. Reusing the identity
with a different type or hash throws `Megaphone.Events.IdentityConflict`.
A unique database constraint handles concurrent calls.

Record events in the same datasource and transaction as your domain change.
`record` doesn't start its own transaction or call a provider. You can configure
storage with constructor `properties.table` and `properties.queryOptions`.

Event identities are separate from inbox entries. Pruning someone's inbox won't
make an old event new again. `find(id)` returns the snapshot or null. Keep event
identities for your supported replay window. Check access before showing their
content; the event store isn't a user inbox. Recording an event doesn't queue or
send deliveries.
