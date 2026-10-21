# Defining Notifications

### Defining a Notification

Notifications are CFCs that extend [`megaphone.models.BaseNotification`](../reference/basenotification.md).

```js
// StockRebalancingCompleteNotification.cfc
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="stockSymbol";
    property name="completionTimestamp";

    public array function via( required any notifiable ) {
        return [ "database" ];
    }

    public struct function toDatabase( required any notifiable ) {
        return {
            "stockSymbol": getStockSymbol(),
            "completionTimestamp": getCompletionTimestamp()
        };
    }

}
```

You can create and set any custom properties you want to store on the notification.

The required methods for you to implement are the [`via`](defining-notifications.md#via) method and any [`to{ChannelType}`](defining-notifications.md#to-channeltype-methods) methods that the notification could be sent to.

### via

The `via` method defines what channels the `Notification` will be sent by returning an array of channel names.  It can return a static array or it can use the passed `Notifiable` instance to dynamically determine the channels.

You may choose certain channels for a certain `Notifiable` type, like only sending SMS messages to `User` instances, not `Team` instances. &#x20;

```js
public array function via( required any notifiable ) {
    return notifiable.getNotifiableType() == "user" ?
        [ "sms", "email" ] :
        [ "email" ];
}
```

You can also store `Notifiable`-specific configuration, like allowing a `User` to opt-in to certain channels like `sms`, `email`, or `slack`.

```js
public array function via( required any notifiable ) {
    return notifiable.getNotificationChannels();
}
```

### to{ChannelType} methods

For each channel type that the `Notification` could be sent on you need to implement a matching `to{ChannelType}` method.&#x20;

{% hint style="info" %}
Note that the method references the Channel Type name, not the Channel name.  If you have configured multiple `DatabaseProvider` channels you would only need one `toDatabase` method.

For example, if your `DatabaseProvider` channel was called `db`, your `via` method would return `[ "db" ]` and you would implement a `toDatabase` method.
{% endhint %}

For instance, if you are using a `DatabaseProvider`, you need a `toDatabase` method.

```js
public array function via( required any notifiable ) {
    return [ "database" ];
}

public struct function toDatabase( required any notifiable ) {
    return {
        "stockSymbol": getStockSymbol(),
        "completionTimestamp": getCompletionTimestamp()
    };
}
```

The `to{ChannelType}` methods also receive the `Notifiable` reference as an argument in case you need to return different data based on the specific `Notifiable` instance.

Each Provider has their own requirements for their `to{ChannelType}` methods.  See the Provider-specific documentation for more information.

### Collecting for a digest

Return one or more [DigestProvider](../providers/digestprovider.md) channels from `via()` and supply their content through `toDigest()`:

```js
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="digests" inject="DigestService@megaphone";
    property name="auditionId";
    property name="publicationEventId";
    property name="publishedAt";

    public array function via( required any notifiable ) {
        return [ "dailyAuditions" ];
    }

    public struct function toDigest( required any notifiable ) {
        return {
            "itemKey": variables.digests.itemKey( [ "audition.published", variables.publicationEventId, 1 ] ),
            "occurredAt": variables.digests.occurredAt( variables.publishedAt ),
            "data": { "auditionId": variables.auditionId }
        };
    }

}
```

Configure `dailyAuditions` with `DigestProvider` and `properties.digestName="dailyAuditions"`. A channel named `weeklyAuditions` using the same Provider also calls `toDigest()`. The Provider type chooses the formatter; the channel's `digestName` chooses the collection.

Use the saved event ID and occurrence date on every retry. `itemKey()` generates a stable key, and `occurredAt()` converts the date to UTC epoch milliseconds. The returned `data` contains serializable values. A missing or stopped subscription suppresses collection; returning a channel does not opt the recipient in.

At the cutoff, Megaphone creates the configured summary Notification with the selected items. That Notification chooses ordinary delivery channels and supplies their formatters. See [Digests](../guides/digests.md) for subscription activation, preference-based routing, and the summary Notification.

### Collecting for an email digest

{% hint style="warning" %}
The email-specific digest API is deprecated in Megaphone 1.3.0. Use [DigestProvider](../providers/digestprovider.md) and [Digests](../guides/digests.md) instead. The new API collects items independently of delivery and publishes a summary Notification that can choose email, SMS, or other channels. The legacy API ties each digest to an email channel and renderer. Existing email digests remain supported for compatibility.
{% endhint %}

The legacy [EmailDigestProvider](../providers/emaildigestprovider.md) calls `toEmailDigest()`. Unlike `toDigest()`, this method includes the named digest in its return value:

```js
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="digests" inject="EmailDigestService@megaphone";
    property name="productId";
    property name="publicationEventId";
    property name="publishedAt";

    public array function via( required any notifiable ) {
        return [ "emailDigest" ];
    }

    public struct function toEmailDigest( required any notifiable ) {
        return {
            "digestName": "productUpdates",
            "itemKey": variables.digests.itemKey( [ "product.published", variables.publicationEventId, 1 ] ),
            "occurredAt": variables.digests.occurredAt( variables.publishedAt ),
            "data": { "productId": variables.productId }
        };
    }

}
```

Configure `emailDigest` with `EmailDigestProvider` and the `productUpdates` definition with an `emailChannel`, policy, and renderer. Use [Named Email Digests](../guides/email-digests.md#choosing-a-delivery-mode) for the legacy subscription API, or the [Upgrade Guide](../upgrade-guide.md#upgrading-to-1.3.0) to migrate.
