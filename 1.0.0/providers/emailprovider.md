# EmailProvider

The `EmailProvider` sends notifications using [cbMailServices](https://forgebox.io/view/cbmailservices).

### Requirements

To use the `EmailProvider`, you need [cbMailServices](https://forgebox.io/view/cbmailservices) installed. This is **not** installed by Megaphone.  If you do not have [cbMailServices](https://forgebox.io/view/cbmailservices) installed, an exception will be thrown if you try to define a channel with the `EmailProvider`.

### Configuration

The `EmailProvider` accepts the following properties:

```js
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

Use [DigestProvider](digestprovider.md) to collect items for a digest. At the cutoff, Megaphone creates the configured summary Notification. Its `via()` method can select an `EmailProvider` channel, and its `toEmail()` method formats the email using the selected items. The same summary can also select other delivery channels. See [Digests](../guides/digests.md#sending-the-summary) for the complete example.

Set the email channel's `properties.durableAdapter` to your receipt-aware adapter. Synchronous cbMailServices success callbacks do not establish durable acceptance. The adapter uses the existing `prepareDelivery(notifiable, notification, delivery)` and `sendDelivery(prepared, delivery)` contract described in [Durable Delivery](../guides/durable-delivery.md).

{% hint style="warning" %}
[EmailDigestProvider](emaildigestprovider.md) is deprecated in Megaphone 1.3.0 in favor of [DigestProvider](digestprovider.md). Its email-channel and renderer API remains supported for compatibility. Use `DigestProvider` for new digests; `EmailProvider` continues to deliver their email summaries.
{% endhint %}

Direct immediate email continues to use this Provider and `toEmail()` normally. An immediate digest instead creates a one-item summary Notification through `DigestProvider`.
