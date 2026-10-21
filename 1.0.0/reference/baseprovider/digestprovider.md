# DigestProvider

`DigestProvider@megaphone` extends `EmailDigestProvider` and uses the collection protocol added in 1.2.0. Its generic digest API requires Megaphone 1.3.0. See [DigestProvider](../../providers/digestprovider.md) for configuration.

### getProviderName

Returns `"DigestProvider"`, independently of the configured channel name.

### getCollectionIntent

`getCollectionIntent(notifiable, notification)` calls `toDigest(notifiable)` and validates `itemKey`, serializable `data`, and numeric `occurredAt` in UTC epoch milliseconds. The Provider requires `properties.digestName` and a named definition with a summary `notification` mapping.

**Return:** The captured item with its canonical `digestName` supplied by channel configuration. This method performs no persistence or transport. An item cannot override the configured name.

**Throws:** `Megaphone.Digests.InvalidDefinition` for missing or incompatible configuration, or `Megaphone.Digests.InvalidItem` for an invalid item or conflicting digest name.

### Inherited collection methods

`supportsCollection()` returns `true`. `notify()` calls the Provider's collection-intent method, so this Provider uses `toDigest()` even when the channel has another name. `collectPlanned(event, intent)` admits a saved item through the current subscription and policy.

The [collection protocol reference](emaildigestprovider.md#collectplanned) describes admission results. For a generic digest, immediate mode creates and dispatches a one-item summary through ordinary notification routing instead of the legacy email renderer. Scheduled mode waits for its cutoff. Collection admissions remain separate from external delivery outcomes.
