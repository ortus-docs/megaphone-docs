# Creating Custom Providers

Providers must extend `megaphone.models.Providers.BaseProvider`.

There are two required methods to implement: `getProviderName` and `notify`.

```js
/**
 * Returns the name for this Provider.
 * This name is the same across all different channels using the same Provider.
 *
 * @return string
 */
public string function getProviderName();

/**
 * Sends a `Notification` to a `Notifiable`.
 * This method will be called once for each `Notifiable` receiving the notification,
 * even if the `NotificationService#notify` method was called with multiple
 * `Notifiable` instances.
 *
 * @notifiable   The `Notifiable` instance receiving the `Notification`.
 * @notification The `Notification` instance to send to the `Notifiable` instance.
 *
 * @return The BaseNotification instance
 */
public BaseNotification function notify(
    required any notifiable,
    required BaseNotification notification
);
```

Inside the `notify` method, there are two helper methods to help you implement your Provider-specific logic: `Notification#routeForType` and `BaseProvider#routeNotificationFor`.

Here is an example using the `EmailProvider`:

```js
public BaseNotification function notify(
    required any notifiable,
    required BaseNotification notification
) {
    var mail = arguments.notification.routeForType(
        type = "email",
        notifiable = notifiable,
        channelName = getName(),
        additionalArgs = {
            "newMail": () => {
                param arguments.mailer = variables.properties.mailer;
                return variables.mailService.newMail( argumentCollection = arguments );
            }
        }
    );

    if ( mail.getProperty( "to" ) == "" ) {
        mail.setProperty( "to", routeNotificationFor( "email", notifiable, getName() ) );
    }

    variables.mailService
        .send( mail )
        .onSuccess( variables.properties.onSuccess )
        .onError( variables.properties.onError );

    return arguments.notification;
}
```

{% hint style="warning" %}
When implementing this method, make sure not to modify the `Notification` instance as this instance will be used for each `Notifiable`.
{% endhint %}

### routeForType

The `routeForType` method is called on the `Notification` instance.  It is what is responsible for calling the `to{ChannelType}` function on the `Notification`.&#x20;

Inside `additionalArgs` you can pass additional parameters that will be passed to the `to{ChannelType}` method after the `Notifiable` instance.

This is a great place to pass helper functions to construct necessary objects for your Provider.  Here in the `EmailProvider`, we provider a `newMail` helper method to create a `Mail` instance from `cbMailServices`. Other Providers provide similar helper functions.

The return value from this function is `any`.  It is the responsibility of your Provider to ensure the return value of this function is what you expected.  For instance, the `DatabaseProvider` expects a struct, the `EmailProvider` expects a `Mail@cbmailservices` instance, and so forth.

```js
var mail = arguments.notification.routeForType(
    type = "email",
    notifiable = notifiable,
    channelName = getName(),
    additionalArgs = {
        "newMail": () => {
            param arguments.mailer = variables.properties.mailer;
            return variables.mailService.newMail( argumentCollection = arguments );
        }
    }
);
```

### routeNotificationFor

The `routeNotificationFor` method is similar to the `routeForType` method.  It is used to ask the `Notifiable` instance to define routing information for a specific Channel type.  For instance, the `EmailProvider` would look for a `routeNotificationForEmail` method.

The `routeNotificationFor{ChannelType}` methods should be defined on the `Notifiable` instance.  The only arguments passed to this function are the `additionalArgs` passed when calling the `routeNotificationFor` methods.

Additionally, the `routeNotificationFor` method can be marked as `optional`.  Optional calls will not throw an exception if they are not defined on the `Notifiable` instance.

The return value from this function is `any`.  It is the responsibility of your Provider to ensure the return value of this function is what you expected.  For instance, the `EmailProvider` expects a `string` value, the `SlackProvider` expects either a `string` value or a `SlackRoute` value.

```js
// EmailProvider.cfc
if ( mail.getProperty( "to" ) == "" ) {
    mail.setProperty( "to", routeNotificationFor( "email", notifiable, getName() ) );
}
```

```js
// SlackProvider.cfc
var route = routeNotificationFor(
    type = "slack",
    notifiable = notifiable,
    channelName = getName(),
    additionalArgs = {
        "newSlackRoute": ( channel, token ) => {
            return variables.wirebox.getInstance(
                "SlackRoute@megaphone",
                { "channel": channel, "token": token }
            );
        }
    },
    optional = true
);
```
