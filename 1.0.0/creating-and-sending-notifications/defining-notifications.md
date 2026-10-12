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
