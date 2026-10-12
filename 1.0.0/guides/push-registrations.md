# Push registrations

`PushSubscriptionService@megaphone.register(notifiable, subscription, label, existingId="", expiresDate, channelName="push")` validates and encrypts browser subscriptions internally. Its public registration metadata excludes endpoints, hashes, and encrypted keys. Use `getRegistrations`, `rename`, `disconnect`, `setExcluded`, `getExcludedIds`, `pruneInactive`, and `deleteNotifiable` for device lifecycle. `retireExpired(notifiable, clock, limit)` retires expired destinations independently; omit the notifiable for bounded maintenance across owners. `getActiveRegistration` is for trusted delivery infrastructure and must not be exposed to clients.

Push channel properties may configure `enabled`, `origin`, `publicKey`, `privateKey`, `subject`, `allowedHosts`, `activeStorageKeyId`, and `storageKeys` directly. The bundled runtime accepts base64 PKCS8/X509 VAPID keys and a storage keyring of base64 AES-256 keys. An optional runtime mapping implements `enabled`, `context(notifiable)`, `cipher`, and `transport`; use it to retain existing recipient/origin encryption contexts. Infrastructure is resolved lazily so disabled or incomplete push does not prevent other channels from starting.

Install the optional Java 11+ SDK from the installed module directory:

```sh
box task run taskFile=tasks/InstallWebPushSDK.cfc
```

The installer verifies the pinned SDK checksum. Enrollment and service-worker integration remain application responsibilities. Registration does not change preferences or publish activity. Disconnect devices on logout/account switch. Provider tests use fixture HTTP clients; verify real devices separately before deployment.
