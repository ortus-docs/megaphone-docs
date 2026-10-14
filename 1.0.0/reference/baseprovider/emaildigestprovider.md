# EmailDigestProvider

`EmailDigestProvider@megaphone` extends `BaseProvider` and implements the optional collection protocol. See [EmailDigestProvider](../../providers/emaildigestprovider.md) for configuration and Notification examples.

### getProviderName

Returns `"EmailDigestProvider"`, independently of the configured channel name.

### notify

`notify(notifiable, notification)` calls the Notification's `toEmailDigest()` method and collects its item using `EmailDigestService`.

**Return:** The populated `BaseNotification`. Collection is synchronous in the caller's transaction; no mail delivery occurs.

### supportsCollection

Returns `true`. `NotificationService.plan()` captures a collection intent instead of requiring a durable transport Provider for this channel.

### getCollectionIntent

`getCollectionIntent(notifiable, notification)` validates the struct returned by `toEmailDigest()`. The required fields are `digestName`, `itemKey`, `data`, and `occurredAt` in UTC epoch milliseconds.

**Return:** The captured item struct. It performs no persistence or transport. An invalid result throws `Megaphone.Digests.InvalidItem`.

### collectPlanned

`collectPlanned(event, intent)` admits one captured item in the publication transaction. It resolves the current Notifiable using `queue.notifiableResolver`, checks the captured recipient identity, and applies the saved channel choice and current subscription/consent policy.

**Return:** A collection admission struct with `id`, `status`, and `created`. `NotificationPublication.getCollections()` exposes these results separately from actual delivery intents.

These collection methods are described by `INotificationCollectionProvider`. They are called by `NotificationService`; applications normally use `notify()`, `queue()`, or `queuePlan()`.
