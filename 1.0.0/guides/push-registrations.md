# Push Registrations

A Web Push subscription belongs to a particular browser on a particular device. `PushSubscriptionService` helps you manage those registrations for a Notifiable, including labels, renewal, and disconnecting a device.

### Registering a browser

Once your browser has created a subscription, send it to your application and register it for the current user:

```js
var registration = pushSubscriptionService.register(
    notifiable = user,
    subscription = subscription,
    label = "My laptop"
);
```

Inject `PushSubscriptionService@megaphone` to get the service. It validates the subscription, hashes its endpoint, encrypts the subscription data, and saves it. The returned registration metadata leaves out endpoints, hashes, and encrypted keys, so you can use it in your device settings screen.

Pass `existingId` when renewing a registration and `expiresDate` when you have an expiry date. If your Web Push channel has a different name, pass it as `channelName`; the default is `push`.

### Managing devices

You can use the same service for your device settings:

| Method | Use it to |
| --- | --- |
| `getRegistrations(notifiable)` | List the Notifiable's active registrations. |
| `rename(notifiable, id, label)` | Change a device's display name. |
| `disconnect(notifiable, id)` | Retire a registration. |
| `setExcluded(notifiable, notificationType, deviceId, excluded)` | Turn a notification type off or on for one device. |
| `getExcludedIds(notifiable, notificationType)` | Read the device exclusions for that notification type. |

Registering a device doesn't change channel preferences or publish a notification. Your application handles browser enrollment and service-worker behavior. It should also disconnect the registration on logout or account switch.

### Configuring subscription storage

The [Web Push Provider](web-push.md) accepts `enabled`, `origin`, `publicKey`, `privateKey`, `subject`, `allowedHosts`, `activeStorageKeyId`, and `storageKeys` in its channel properties.

The bundled runtime uses base64 PKCS8/X509 VAPID keys and a storage keyring of base64 AES-256 keys. If your application already encrypts subscriptions, you can supply a `runtime` mapping with `enabled()`, `context(notifiable)`, `cipher()`, and `transport()`. Keep the existing recipient and origin contexts so saved subscriptions can still be decrypted.

Push infrastructure is resolved when needed. A disabled channel or incomplete push configuration won't prevent unrelated channels from starting.

### Retiring old registrations

`retireExpired(notifiable, clock, limit)` retires expired registrations independently. For scheduled maintenance across owners, omit the Notifiable and use a bounded limit. `pruneInactive(beforeDate, limit)` removes old inactive registrations, and `deleteNotifiable(notifiable)` removes registrations when you clean up an account.

{% hint style="warning" %}
`getActiveRegistration()` exposes data needed by trusted delivery code. Use `getRegistrations()` for your client-facing device list instead.
{% endhint %}
