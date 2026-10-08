# Notification preferences

`PreferenceResolver@megaphone.resolve(defaults, scopes)` takes channel defaults
and scopes ordered from least to most specific. Each scope has a `key` and a
`choices` struct with `inherit`, `on`, or `off` for each channel. The result gives
you the effective `enabled` value and the `source` key for each channel. Your
inputs aren't changed. You choose the scope names and check access to them.
If you want to store those choices, apply the preference migration and use
`PreferenceStore@megaphone`.

`selectDevices(devices, excludedIds)` picks active registrations by stable ID and
removes duplicates and excluded devices. New registrations are included unless
excluded. Enrollment, storage, and sending happen outside this resolver.

`PreferenceStore.saveChoice(notifiable, notificationType, scopeKey, channel,
choice)` saves `on` or `off`; `inherit` removes that recipient's exact choice.
`choices(notifiable, notificationType, scopeKeys)` returns scopes in the order
you supplied, including empty choices for inherited scopes.
`configuration(notifiable, notificationType, defaults, scopeKeys)` returns both
the scopes and the resolved values. Check that the caller can use the requested
notification types and scopes before calling these methods. A scope string
isn't an authorization check. `deleteRecipient(notifiable)` removes only that
recipient's choices.

`PreferenceStore.pruneChoices(constraints, limit=100)` removes a batch of stale
choices across recipients. Supply a backend QueryBuilder callback that selects
obsolete scopes and keeps live scopes and account defaults. Megaphone doesn't
know which application resources still exist. Candidates use a deterministic
composite-key order and are checked again under row locks before deletion.
`limit` must be an integer from 1 to 1,000. Keep this as a maintenance operation;
don't let a user supply an arbitrary query or scope deletion.
