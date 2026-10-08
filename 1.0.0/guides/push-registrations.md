# Push registrations

Apply the subscriptions and device-exclusions migrations before using
`SubscriptionStore@megaphone`. `register(notifiable, endpointHash, sealedData,
label, existingId, expiresDate)` takes a SHA-256 endpoint hash and encrypted
subscription data. Pass `existingId` when renewing that recipient's registration
to preserve its device exclusions. When a subscription moves to another account,
the old registration is retired, its encrypted data is cleared, and the new
account gets a separate registration ID.

`registrations(notifiable, activeOnly=true, clock=now())` returns device metadata
without endpoints, hashes, or encrypted keys. Use
`activeRegistration(notifiable, id, clock)` internally when sending to an active,
unexpired registration. Keep that result out of client responses and logs.
`rename` and `disconnect` return false for a missing ID or another recipient's ID.
Use `setExcluded(notifiable, notificationType, deviceId, excluded)` and
`excludedIds(notifiable, notificationType)` for per-type device exclusions.
`pruneInactive(beforeDate, limit=100)` removes retired registrations and their
exclusions in batches. `deleteRecipient` removes that recipient's records.

Disconnect registrations when the user logs out or switches accounts. You'll
need to wire this into your app; Megaphone doesn't listen to authentication
events automatically.

The active list excludes expired subscriptions using the same clock as dispatch.
Pass `activeOnly=false` to show expired and disconnected registrations on a
device-management screen. A stored active flag doesn't override an expiry date.

`SubscriptionCipher(activeKeyId, keys)` encrypts JSON subscription data with
AES-256-GCM. Each key is a base64-encoded 32-byte storage key. Pass the same
nonempty recipient-and-origin context to `seal(subscription, context)` and
`unseal(sealedData, context)`. Keep old key IDs while records still use them;
new writes use the active key. These storage keys are separate from your VAPID
keys. Validate the browser's subscription fields in your enrollment endpoint.

`WebPushResponsePolicy.classify(statusCode, retryAfter="", clock=now())` returns
a delivery outcome, a safe reason code, and `retireSubscription`. A 201 or 202
means the push service accepted the request; it doesn't tell you whether the
user received it. A 404 or 410 retires the registration. A 429 or server error
can be retried with a bounded retry date. Timeouts, redirects, and unexpected
success codes leave acceptance ambiguous. Response bodies and endpoint URLs
aren't retained.

Use the classifier when you have a completed HTTP response. If transport throws
after sending, keep that result ambiguous through `DeliveryDispatcher`.
