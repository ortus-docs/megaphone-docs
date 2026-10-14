# Sending Notifications

There are two ways to send notifications in Megaphone — using the [`NotificationService`](sending-notifications.md#notificationservice) or using [delegates](sending-notifications.md#sendsnotifications-delegate) (requires ColdBox 7+).

### NotificationService

Notifications are sent using the [`NotificationService`](../reference/notificationservice.md), often aliased as `megaphone`.

```js
// handlers/StockRebalancing.cfc
component {

    property name="megaphone" inject="NotificationService@megaphone";

    function create( event, rc, prc ) {
        // ...
        var notification = getInstance( "StockRebalancingCompleteNotification" )
        notification.setStockSymbol( "APPL" );
        notification.setCompletionTimestamp( now() );
        megaphone.notify( auth().user(), notification );
        // ...
    }

}
```

For those of you allergic to calling `getInstance` (😜), you can also pass a string name and a struct of properties:

```js
// handlers/StockRebalancing.cfc
component {

    property name="megaphone" inject="NotificationService@megaphone";

    function create( event, rc, prc ) {
        // ...
        megaphone.notify(
            auth().user(),
            "StockRebalancingCompleteNotification",
            { "stockSymbol": "APPL", "completionTimestamp": now() }
        );
        // ...
    }

}
```

### SendsNotifications Delegate

Another way to send a Notification is by adding the [`SendsNotifications`](../reference/sendsnotifications.md) delegate to a [`Notifiable`](../reference/inotifiable.md) instance.

```js
component name="User" delegates="SendsNotifications@megaphone" accessors="true" {

    property name="id";

    public string function getNotifiableId() {
        return getId();
    }

    public string function getNotifiableType() {
        return "User";
    }

}
```

Then you can call a `notify` method on the [`Notifiable`](../reference/inotifiable.md) instance.

```js
// handlers/StockRebalancing.cfc
component {

    property name="megaphone" inject="NotificationService@megaphone";

    function create( event, rc, prc ) {
        // ...
        auth().user().notify(
            "StockRebalancingCompleteNotification",
            { "stockSymbol": "APPL", "completionTimestamp": now() }
        );
        // ...
    }

}
```

{% hint style="info" %}
The `notify` method from the delegate can be passed either a [`Notification`](../reference/basenotification.md) instance or a string name just like the `notify` method on the [`NotificationService`](../reference/notificationservice.md).
{% endhint %}
