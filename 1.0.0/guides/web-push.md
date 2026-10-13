# Java Web Push

The `WebPushProvider` sends a Notification to a Notifiable's registered browsers. Like the other Providers, you register it as a channel and return that channel name from `via()`.

### Requirements

The bundled transport needs Java 11+. Megaphone declares the Web Push SDK as a pinned JAR dependency in `box.json`. CommandBox installs it with the module, so there's no separate SDK installation or build step.

The SDK uses the JDK's HTTP client and needs no additional Java libraries or fat JAR. Megaphone verifies its checksum and loads it when you use Web Push; other channels don't require Java 11 or push configuration.

### Configuration

Register the Provider through your Megaphone channel settings:

```js
"push": {
    "provider": "WebPushProvider@megaphone",
    "properties": {
        "runtime": "ApplicationPushRuntime"
    }
}
```

Here, `ApplicationPushRuntime` is your application component supplying the push configuration, cipher, and transport. You can also configure the bundled runtime directly through channel properties. See [Push Registrations](push-registrations.md#configuring-subscription-storage) for those settings and encrypted subscription storage.

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

For browser enrollment, renewal, and logout behavior, see [Push Registrations](push-registrations.md). The module tests use fixture HTTP clients; check your actual Provider configuration and devices before deploying your application's push integration.
