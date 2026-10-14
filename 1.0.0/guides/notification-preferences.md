# Notification Preferences

A user may want email for one kind of notification and only an inbox entry for another. They may also want different choices for different teams or projects. `NotificationPreferenceService` stores those choices while letting your application decide what each scope means.

### Where preferences are stored

Preferences live in the `megaphone_preferences` database table, created by Megaphone's module migrations. Apply those migrations through your application's migration ledger before using the service. Preferences are separate from the inbox entries stored by the Database Provider, and work with synchronous Notifications as well as queued Notifications.

Each stored choice belongs to a Notifiable and identifies a notification type, scope key, and channel. For example, one row can say that User 123 has turned email off for `OrderReadyNotification` in `team:42`. Defaults aren't stored as rows, and choosing `inherit` removes that particular override.

By default, the store uses your application's default datasource. You can choose a datasource or table through `moduleSettings.megaphone.storage.preferences`:

```js
moduleSettings.megaphone.storage = {
    "preferences": {
        "table": "app_notification_preferences",
        "datasource": "app"
    }
};
```

The store also accepts `queryOptions`. A `datasource` property takes precedence over `queryOptions.datasource`. If you customize the table name, create the same preference schema in that table; changing the setting doesn't rename the table created by the supplied migration.

### Saving a preference

Inject `NotificationPreferenceService@megaphone` into the component that manages your user's settings:

```js
property name="notificationPreferenceService"
    inject="NotificationPreferenceService@megaphone";
```

Pass the Notifiable, notification type, scope key, and channel name. Here, the user turns email off across their account, then turns it back on for one team:

```js
notificationPreferenceService.setPreference(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKey = "account",
    channelName = "email",
    choice = "off"
);

notificationPreferenceService.setPreference(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKey = "team:42",
    channelName = "email",
    choice = "on"
);
```

The choice can be `on`, `off`, or `inherit`. Scope keys are strings your application defines. `account` and `team:42` have no special meaning to Megaphone; their order when you resolve them determines which choice wins.

Use the same notification type when saving and resolving preferences, and use your configured channel names. If your email channel is named `transactionalEmail`, use that name in both the preference and the Notification's `via()` method.

### Resolving scopes

Your application supplies the defaults and scope keys, ordered from broadest to most specific:

```js
var preferences = notificationPreferenceService.resolvePreferences(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    defaults = { "database": true, "email": true },
    scopeKeys = [ "account", "team:42" ]
);
var effective = preferences.effective;
```

With the choices above, `effective` contains:

```js
{
    "database": { "enabled": true, "source": "default" },
    "email": { "enabled": true, "source": "team:42" }
}
```

The team choice takes precedence over the account choice. The returned struct also contains `scopes`, with the explicit choices. Each channel resolves independently, so changing email doesn't change the inbox preference.

You can include more scopes, such as `[ "account", "team:42", "project:7" ]`. Put them in order from broadest to most specific; the last explicit choice for each channel wins. With no stored choice, that channel uses the default you supplied.

Megaphone owns the Notifiable's preference records. Your application still needs to check whether the user can view or change each scope.

### Using preferences in a Notification

Override `getChannelPreferences(notifiable)` on your [Notification](../reference/basenotification.md#getchannelpreferences) to apply these effective values. The hook returns a map keyed by the configured channel name, with `{ enabled, source }` for each channel from `via()`.

Here's an `OrderReadyNotification` that sends an inbox entry and an email, subject to the user's account and team choices:

```js
// models/notifications/OrderReadyNotification.cfc
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="orderId";
    property name="teamId";
    property name="notificationPreferenceService"
        inject="NotificationPreferenceService@megaphone";

    public string function getNotificationType() {
        return "OrderReadyNotification";
    }

    public array function via( required any notifiable ) {
        return [ "database", "email" ];
    }

    public struct function getChannelPreferences(
        required any notifiable,
        array channelNames
    ) {
        var preferences = variables.notificationPreferenceService.resolvePreferences(
            notifiable = arguments.notifiable,
            notificationType = getNotificationType(),
            defaults = { "database": true, "email": true },
            scopeKeys = [ "account", "team:#variables.teamId#" ]
        );
        return preferences.effective;
    }

    public struct function toDatabase( required any notifiable ) {
        return {
            "orderId": variables.orderId,
            "teamId": variables.teamId,
            "message": "Your order is ready."
        };
    }

    public any function toEmail( required any notifiable, required any newMail ) {
        return arguments.newMail(
            to = arguments.notifiable.getEmail(),
            from = "orders@example.com",
            subject = "Your order is ready",
            type = "text"
        ).setBody( "Order #variables.orderId# is ready for collection." );
    }

}
```

`channelNames` is an optional argument Megaphone supplies during planning and synchronous delivery. This Notification always offers the same two channels, so it can use the same defaults each time. Include a preference result for every channel your `via()` method returns.

With `database` and `email` channels configured, send the Notification as usual:

```js
megaphone.notify(
    notifiables = user,
    notification = "OrderReadyNotification",
    properties = { "orderId": "1042", "teamId": "42" }
);
```

For team 42, both channels are enabled by the choices we saved above. For another team with no override, the account choice disables email and only the inbox entry is sent.

Saving a preference alone doesn't change a Notification's behavior. The default `getChannelPreferences()` enables every channel from `via()`; your override connects the Notification to the stored choices.

`notify()` skips disabled channels. `plan()` captures those choices, and `queue()` records disabled channels as suppressed deliveries. Turning a channel on later doesn't send previously suppressed notifications. A worker also checks current preferences before sending an enabled delivery, so a user can opt out while it is waiting. See [Durable Delivery](durable-delivery.md) for queue setup, including the durable adapter required by the Email Provider.

### Resetting a choice to inherit

If the user wants team 42 to follow their account choice again, remove the team override with `inherit`:

```js
notificationPreferenceService.setPreference(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKey = "team:42",
    channelName = "email",
    choice = "inherit"
);
```

Email now resolves to `{ "enabled": false, "source": "account" }`. Resetting the account choice to `inherit` too would restore the Notification's default of `true`.

### Showing choices in a settings screen

Use `getPreferences()` when you need the saved overrides without applying defaults:

```js
var scopes = notificationPreferenceService.getPreferences(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKeys = [ "account", "team:42" ]
);
```

After resetting the team choice above, the result is:

```js
[
    { "key": "account", "choices": { "email": "off" } },
    { "key": "team:42", "choices": {} }
]
```

An absent channel choice means `inherit`. If your screen should also show whether the channel is currently enabled and where that decision came from, use `resolvePreferences()` and display its `effective` values alongside `scopes`.

### Importing and cleaning up

Use `importPreference()` with the same arguments as `setPreference()` to bring in choices from another source without replacing an explicit choice already stored:

```js
notificationPreferenceService.importPreference(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKey = "account",
    channelName = "email",
    choice = "off"
);
```

`prunePreferences(constraints, limit)` removes a bounded batch selected by your application's maintenance constraints. When you remove an account, delete that Notifiable's preferences too:

```js
notificationPreferenceService.deleteNotifiable( user );
```
