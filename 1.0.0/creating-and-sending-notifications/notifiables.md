# Notifiables

A `Notifiable` is a something that can recieve a notification.  Traditionally it is your `User` object, but it isn't limited to that.  You may send notifications to a `Team`, a `MailingList`, a `Site`, or more.

### INotifiable

A `Notifiable` needs to implement the `INotifiable` interface (`implements` keyword optional).

```js
interface displayName="INotifiable" {

    /**
     * The id representing this notifiable.
     */
    public string function getNotifiableId();

    /**
     * The type name representing this notifiable.
     */
    public string function getNotifiableType();

}
```

An example implementation for a `User` component could be as follows:

```js
component
    name="User"
    accessors="true"
    implements="megaphone.models.Interfaces.INotifiable"
{

    property name="id";
    
    /**
     * The id representing this notifiable.
     */
    public string function getNotifiableId() {
        return getId();
    }

    /**
     * The type name representing this notifiable.
     */
    public string function getNotifiableType() {
        return "user";
    }

}
```

### via

`Notifiable` instances are passed to the `via` method on a `Notification`. This is to allow you to customize the channels used to each `Notifiable`.  You may choose certain channels for a certain `Notifiable` type, like only sending SMS messages to `User` instances, not `Team` instances.  You can also store `Notifiable`-specific configuration, like allowing a `User` to opt-in to certain channels like `sms`, `email`, or `slack`.

{% hint style="info" %}
See the [`via` docs on `Notifications`](defining-notifications.md#via) for more information.
{% endhint %}

### routeNotificationFor

Providers may look for a `routeNotificationFor` method suffixed with the Provider type name.  For instance, the `EmailProvider` may look for a `routeNotificationForEmail` method on the `Notifiable`.&#x20;

{% hint style="info" %}
See the Provider-specific docs for more information.
{% endhint %}
