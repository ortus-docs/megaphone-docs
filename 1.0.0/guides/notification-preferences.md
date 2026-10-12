# Notification Preferences

A user may want email for one kind of notification and only an inbox entry for another. They may also want different choices for different teams or projects. `NotificationPreferenceService` stores those choices while letting your application decide what each scope means.

### Saving a preference

Inject `NotificationPreferenceService@megaphone` and pass the Notifiable, notification type, scope key, and channel name:

```js
notificationPreferenceService.setPreference(
    notifiable = user,
    notificationType = "OrderReadyNotification",
    scopeKey = "team:42",
    channelName = "email",
    choice = "off"
);
```

The choice can be `on`, `off`, or `inherit`. `inherit` removes the override for that exact scope and channel, so the broader preference or default applies again. Megaphone only stores explicit overrides.

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

In this example, a choice for `team:42` takes precedence over an account-level choice. The returned struct contains `scopes`, with the explicit choices, and `effective`, with the resolved values. Use `getPreferences(notifiable, notificationType, scopeKeys)` if you only need the stored choices.

Megaphone owns the Notifiable's preference records. Your application still needs to check whether the user can view or change each scope.

### Using preferences in a Notification

Override `getChannelPreferences(notifiable)` on your [Notification](../reference/basenotification.md#getchannelpreferences) to apply these effective values. The hook returns a map keyed by the configured channel name, with `{ enabled, source }` for each channel from `via()`.

`notify()` skips disabled channels. `plan()` captures those choices, and `queue()` records disabled channels as suppressed deliveries. Turning a channel on later doesn't send previously suppressed notifications.

### Importing and cleaning up

Use `importPreference()` with the same arguments as `setPreference()` to bring in existing choices without replacing an explicit choice already stored. `prunePreferences(constraints, limit)` removes a bounded batch selected by your application's maintenance constraints. `deleteNotifiable(notifiable)` removes that Notifiable's preferences when you are cleaning up their account.
