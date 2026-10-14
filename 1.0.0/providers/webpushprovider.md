# WebPushProvider

The `WebPushProvider` sends a Notification to a Notifiable's registered browsers. Like the other Providers, you register it as a channel and return that channel name from `via()`.

### Requirements

Use Megaphone's JDK 11+ baseline for the bundled transport. Megaphone declares the Web Push SDK as a pinned JAR dependency in `box.json`. CommandBox installs it with the module, so there's no separate SDK installation or build step.

The SDK uses the JDK's HTTP client and needs no additional Java libraries or fat JAR. Megaphone verifies its checksum and loads it when you use Web Push; other channels don't require push configuration.

On Adobe ColdFusion, add the installed SDK folder to your application's Java classpath in `Application.cfc`, alongside any existing `loadPaths`:

```js
this.javaSettings = {
    "loadPaths": [ "./modules/megaphone/resources/java/webpush" ],
    "loadColdFusionClassPath": true
};
```

Adjust the path if your application installs modules elsewhere. Megaphone still verifies the SDK checksum before using it.

### Configuration

Megaphone includes a default `WebPushRuntime`. It supplies the subscription cipher, recipient context, browser enrollment public key, and Java transport. You don't need to create an `ApplicationPushRuntime` component.

Register the Provider through your Megaphone channel settings and supply your application's keys and push configuration:

```js
moduleSettings.megaphone.channels = {
    "push": {
        "provider": "WebPushProvider@megaphone",
        "properties": {
            "enabled": true,
            "origin": "https://example.com",
            "subject": "mailto:push@example.com",
            "publicKey": getSystemSetting( "PUSH_VAPID_PUBLIC_KEY" ),
            "privateKey": getSystemSetting( "PUSH_VAPID_PRIVATE_KEY" ),
            "allowedHosts": listToArray( getSystemSetting( "PUSH_ALLOWED_HOSTS" ) ),
            "activeStorageKeyId": "v1",
            "storageKeys": {
                "v1": getSystemSetting( "PUSH_STORAGE_KEY" )
            }
        }
    }
};
```

Leave out `runtime` to use Megaphone's default. The environment variable names above are examples; use your application's configuration conventions.

`publicKey` is a base64 X509 VAPID public key, and `privateKey` is its matching base64 PKCS8 private key. `storageKeys` contains base64 AES-256 keys for encrypting subscriptions, with `activeStorageKeyId` selecting the key used for new writes. `allowedHosts` lists the trusted push-service hosts your application sends to. Keep the configured keys and `origin` stable across restarts so existing subscriptions remain usable.

The default encryption context combines the Notifiable's type, ID, and configured origin. See [Push Registrations](../guides/push-registrations.md) for browser enrollment, encrypted storage, and key rotation. Push infrastructure is resolved when needed; setting `enabled` to `false` disables push without preventing unrelated channels from starting.

### toWebPush

Your Notification implements `toWebPush` to return the push payload:

```js
public array function via( required any notifiable ) {
    return [ "push" ];
}

public struct function toWebPush( required any notifiable, required any delivery ) {
    return {
        "title": "Your order is ready",
        "body": "You can pick it up at the front desk."
    };
}
```

The Provider passes the Notifiable and delivery to `toWebPush`, so you can format the content for that delivery. Your application's service worker decides how to display the payload.

### routeNotificationForWebPush

By default, Megaphone uses the Notifiable's owned, active registrations. You can add `routeNotificationForWebPush()` to the Notifiable to return an array of device metadata with registration IDs. Megaphone still checks that those registrations belong to the Notifiable and applies device exclusions for the notification type.

For durable notifications, each selected device gets its own delivery. If no device is eligible, Megaphone records suppressed work without making a transport request. An expired registration is retired independently, so one expired device doesn't cancel deliveries to other devices.

The Provider handles encryption, request preparation, and HTTP outcomes behind its normal delivery methods. Your Notification only needs to supply the content.

For browser enrollment, renewal, and logout behavior, see [Push Registrations](../guides/push-registrations.md). The module tests use fixture HTTP clients; check your actual Provider configuration and devices before deploying your application's push integration.

### Providing a custom runtime

If your application needs a different encryption context, cipher, or transport, you can override the default with a WireBox mapping:

```js
"push": {
    "provider": "WebPushProvider@megaphone",
    "properties": {
        "runtime": "ApplicationPushRuntime"
    }
}
```

`ApplicationPushRuntime` is your component in this example. It implements `enabled()`, `context(notifiable)`, `cipher()`, and `transport()`. Use this override when integrating existing push infrastructure or preserving the encryption context of subscriptions already stored by your application. Otherwise, use the default runtime and configure its channel properties.
