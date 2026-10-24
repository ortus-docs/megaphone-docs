# Push Registrations

A Web Push subscription belongs to a particular browser on a particular device. `PushSubscriptionService` helps you manage those registrations for a Notifiable, including labels, renewal, and disconnecting a device.

### Getting a subscription from the browser

The `subscription` comes from the browser's [Push API](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/subscribe). Register a service worker, then call `registration.pushManager.subscribe()` when the user chooses to enable push. This needs HTTPS, or localhost during development, and notification permission.

You'll also need an `applicationServerKey`: the browser-compatible VAPID public key belonging to the key pair your [Web Push Provider](../providers/webpushprovider.md) uses. Keep the private key on the server. The bundled runtime's `enrollmentConfiguration()` method returns this public key in the right format:

```js
// Server-side, when using the bundled runtime's channel properties.
var pushChannel = wirebox.getInstance( "megaphone:push" );
var enrollment = wirebox.getInstance(
    "WebPushRuntime@megaphone",
    { "properties": pushChannel.getProperties() }
).enrollmentConfiguration();
```

Check `enrollment.available` before offering enrollment, then pass `enrollment.applicationServerKey` to your page. If you use a custom runtime, have your application supply the corresponding browser-compatible public key. The bundled runtime converts the configured X509 public key for you; don't pass that encoded server key directly to the browser.

Here's a browser example. Run the setup in a JavaScript module, then call `enablePush()` from an "Enable notifications" button after the service worker is ready:

```js
await navigator.serviceWorker.register( "/sw.js" );
const registration = await navigator.serviceWorker.ready;

async function enablePush( applicationServerKey, label, csrfToken ) {
    const permission = await Notification.requestPermission();
    if ( permission !== "granted" ) {
        throw new Error( "Allow notifications to enable push." );
    }

    let subscription = await registration.pushManager.getSubscription();
    if ( !subscription ) {
        subscription = await registration.pushManager.subscribe( {
            userVisibleOnly: true,
            applicationServerKey
        } );
    }

    const response = await fetch( "/account/push-registrations", {
        method: "POST",
        credentials: "same-origin",
        headers: {
            "Content-Type": "application/json",
            "X-CSRF-Token": csrfToken
        },
        body: JSON.stringify( {
            subscription: subscription.toJSON(),
            label
        } )
    } );

    if ( !response.ok ) {
        throw new Error( "Could not save this browser's push registration." );
    }
    return response.json();
}
```

The URL and CSRF header are examples for your application to implement. Use your usual authenticated request handling and show errors from `enablePush()` in the settings screen. Offer the button only when the browser supports service workers, `PushManager`, and `Notification`.

[`getSubscription()`](https://developer.mozilla.org/en-US/docs/Web/API/PushManager/getSubscription) reuses an existing subscription for this service worker. [`toJSON()`](https://developer.mozilla.org/en-US/docs/Web/API/PushSubscription/toJSON) supplies the endpoint, expiry time, and keys that you send to the server. You don't need to assemble those values yourself.

### Receiving push messages

Create `/sw.js` in your application's web root. For the `title` and `body` payload shown in [WebPushProvider](../providers/webpushprovider.md#towebpush), a minimal service worker looks like this:

```js
self.addEventListener( "push", ( event ) => {
    const message = event.data ? event.data.json() : {};
    event.waitUntil( self.registration.showNotification(
        message.title || "New notification",
        { body: message.body || "" }
    ) );
} );
```

If your application already has a service worker, add the handler there and reuse its registration. Your application can also handle notification clicks and navigation. See MDN's [service worker registration](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerContainer/register) and [push event](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope/push_event) documentation for the browser APIs.

### Saving the subscription in Megaphone

In your application's POST handler, read the JSON request body and pass its `subscription` struct to Megaphone. Load `user` from the authenticated session:

```js
var payload = event.getHTTPContent( json = true );
var registration = pushSubscriptionService.register(
    notifiable = user,
    subscription = payload.subscription,
    label = payload.label
);
```

Return `registration` as JSON to complete the browser request above. Megaphone supplies the registration service; your application supplies the HTTP endpoint and identifies the current user.

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
| `getExcludedIdsMany(notifiable, notificationTypes)` | Read exclusions for several notification types in one batch. Available in the upcoming release. |

Registering a device doesn't change channel preferences or publish a notification. Your application handles browser enrollment and service-worker behavior. It should also disconnect the registration on logout or account switch.

### Reading exclusions for several notification types

{% hint style="info" %}
`getExcludedIdsMany()` is available in the upcoming release after 1.3.0. See [the module change](https://github.com/coldbox-modules/megaphone/pull/6).
{% endhint %}

A notification settings page may need the device exclusions for every type it displays. Read them together instead of calling `getExcludedIds()` inside a loop:

```js
var notificationTypes = [ "OrderReadyNotification", "InvoicePaidNotification" ];
var exclusions = pushSubscriptionService.getExcludedIdsMany(
    notifiable = user,
    notificationTypes = notificationTypes
);

// exclusions[ 1 ] contains device IDs excluded for OrderReadyNotification.
// exclusions[ 2 ] contains device IDs excluded for InvoicePaidNotification.
```

Load `user` from the authenticated session, as in the registration example. Your application still authorizes which notification types the user can view or change.

| Name | Type | Required | Description |
| --- | --- | --- | --- |
| `notifiable` | `INotifiable` | true | The owner of the exclusions. Reads are constrained by both recipient type and ID. |
| `notificationTypes` | `array` | true | Notification-type strings in the order you want the results. |

**Return:** An array of device-ID arrays, with one entry for each input type. Each entry matches `getExcludedIds(notifiable, notificationType)`. Types without exclusions return an empty array. Repeated types return separate arrays, so changing one result does not change another.

The stock store uses Megaphone's existing bounded identity reader, with at most 100 requested types per query. Forty types require one query; 101 require two. Comparisons follow the database's equality rules, including its collation. The batch uses the same configured subscription storage options as the scalar lookup.

Pass nonblank notification-type strings no longer than 160 characters. Invalid entries raise `Megaphone.Push.InvalidExclusion` before any queries run. An empty `notificationTypes` array returns an empty array without querying, for a valid Notifiable.

Each call reads current exclusions. The method does not cache preferences or device state, and reading exclusions does not change a device or its delivery preferences.

### Configuring subscription storage

The [Web Push Provider](../providers/webpushprovider.md#configuration) uses Megaphone's default `WebPushRuntime` when you omit `runtime`. Configure it with `enabled`, `origin`, `publicKey`, `privateKey`, `subject`, `allowedHosts`, `activeStorageKeyId`, and `storageKeys` in its channel properties.

The bundled runtime uses base64 PKCS8/X509 VAPID keys and a storage keyring of base64 AES-256 keys. If your application already encrypts subscriptions, you can supply a `runtime` mapping with `enabled()`, `context(notifiable)`, `cipher()`, and `transport()`. Keep the existing recipient and origin contexts so saved subscriptions can still be decrypted.

To rotate a subscription storage key, add a new key to `storageKeys` and select its ID with `activeStorageKeyId`. Keep the old keys while saved subscriptions still use them. Changing the active storage key doesn't change the VAPID key pair used for browser enrollment.

Push infrastructure is resolved when needed. A disabled channel or incomplete push configuration won't prevent unrelated channels from starting.

### Retiring old registrations

`retireExpired(notifiable, clock, limit)` retires expired registrations independently. For scheduled maintenance across owners, omit the Notifiable and use a bounded limit. `pruneInactive(beforeDate, limit)` removes old inactive registrations, and `deleteNotifiable(notifiable)` removes registrations when you clean up an account.

{% hint style="warning" %}
`getActiveRegistration()` exposes data needed by trusted delivery code. Use `getRegistrations()` for your client-facing device list instead.
{% endhint %}
