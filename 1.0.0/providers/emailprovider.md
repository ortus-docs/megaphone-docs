# EmailProvider

The `EmailProvider` sends notifications using [cbMailServices](https://forgebox.io/view/cbmailservices).

### Requirements

To use the `EmailProvider`, you need [cbMailServices](https://forgebox.io/view/cbmailservices) installed. This is **not** installed by Megaphone.  If you do not have [cbMailServices](https://forgebox.io/view/cbmailservices) installed, an exception will be thrown if you try to define a channel with the `EmailProvider`.

### Configuration

The `EmailProvider` accepts the following properties:

```json
{
    "mailer": "default", // optional, uses the cbMailServices default otherwise
    "onSuccess": () => {}, // optional, logs to info otherwise
    "onFailure": () => {} // optional, logs to error otherwise
}
```

### toEmail

The `toEmail` method returns a `Mail@cbmailservices` instance to send. The `to` property can be either defined in the `toEmail` method or a `routeNotificationForEmail` method defined on the `Notifiable` instance.

```js
public struct function toEmail( notifiable, newMail ) {
    return newMail(
        to = notifiable.getEmail(),
        from = "noreply@example.com",
        subject = "Megaphone Email Notification",
        type = "html",
        bodyTokens = { product: "ColdBox" }
    ).setBody( "
        <p>Thank you for downloading @product@, have a great day!</p>
    " );
}
```

### routeNotificationForEmail

You can let the `Notifiable` instance define how to send the Email Notification by adding a `routeNotificationForEmail` method.

```js
component accessors="true" {
    
    property name="email";
    
    public string function routeNotificationForEmail() {
        return getEmail();
    }

}
```

### Sending a digest

[EmailDigestProvider](emaildigestprovider.md) collects named digest items. The due dispatcher sends the rendered summary through a configured email channel. Set that channel's `properties.durableAdapter` to your receipt-aware adapter; synchronous cbMailServices success callbacks do not establish durable acceptance.

The adapter uses the existing `prepareDelivery(notifiable, notification, delivery)` and `sendDelivery(prepared, delivery)` contract described in [Durable Delivery](../guides/durable-delivery.md). Immediate email continues to use this Provider and `toEmail()` normally.
