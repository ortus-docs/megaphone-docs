# Configuration

Like cbmailservices and cbfs, Megaphone creates channels to send `Notification` instances on.  Channels are created by defining `provider` and any necessary `properties`.

```js
// config/modules/megaphone.cfc
component {

    function configure() {
        return {
            "channels": {
                // this is the unique name for the channel
                "database": {
                    "provider": "DatabaseProvider@megaphone",
                    "properties": {
                        // this is the default table name
                        "table": "megaphone_notifications",
                        "datasource": "megaphone"
                    }
                },
                "db2": {
                    // providers can be used multiple times with different properties
                    "provider": "DatabaseProvider@megaphone",
                    "properties": {
                        "datasource": "db2"
                    }
                },
                "email": {
                    "provider": "EmailProvider@megaphone",
                    "properties": {
                        "mailer": "default",
                        "onSuccess": () => {},
                        "onError": () => {}
                    }
                },
                "slack": {
                    "provider": "SlackProvider@megaphone",
                    "properties": {
                        "token": getSystemSetting( "SLACK_BOT_TOKEN" ),
                        "defaultChannel": "##general"
                    }
                }
            }
        };
    }

}
```

Each Provider defines the required and optional `properties` it accepts.  Refer to the individual Provider documentation for more information.

### Digests

[DigestProvider](../providers/digestprovider.md) collects items for a named digest. Each channel supplies `properties.digestName`. The definition supplies a policy, schedule, and summary Notification mapping:

```js
moduleSettings.megaphone.channels.dailyAuditions = {
    "provider": "DigestProvider@megaphone",
    "properties": { "digestName": "dailyAuditions" }
};

moduleSettings.megaphone.digests.dailyAuditions = {
    "schedule": "everyDayAt 09:00",
    "timezone": "America/Denver",
    "policy": "AuditionDigestPolicy",
    "notification": "DailyAuditionsNotification"
};
```

The source Notification supplies `toDigest()`. At the cutoff, `DailyAuditionsNotification` receives the selected items and chooses its own delivery channels through `via()`. Keep your ordinary email, SMS, or other channel registrations. The digest definition does not need an `emailChannel` or `renderer`.

Storage remains under `storage.digests` and shares the transactional datasource used by events and deliveries. Subscriptions require explicit activation. See [Digests](../guides/digests.md#configuring-a-digest) for the complete daily and weekly setup.

### Named email digests

{% hint style="warning" %}
The email-specific digest API is deprecated in Megaphone 1.3.0. Use [DigestProvider](../providers/digestprovider.md) and [Digests](../guides/digests.md) instead. The new API collects items independently of delivery and publishes a summary Notification that can choose email, SMS, or other channels. The legacy API ties each digest to an email channel and renderer. Existing email digests remain supported for compatibility.
{% endhint %}

[EmailDigestProvider](../providers/emaildigestprovider.md) uses `toEmailDigest()` to choose the named digest. Each legacy definition selects an email channel and renderer:

```js
moduleSettings.megaphone.channels.email = {
    "provider": "EmailProvider@megaphone",
    "properties": { "durableAdapter": "ReceiptAwareEmailAdapter" }
};

moduleSettings.megaphone.channels.emailDigest = {
    "provider": "EmailDigestProvider@megaphone",
    "properties": {}
};

moduleSettings.megaphone.digests.productUpdates = {
    "schedule": "everyWeekOn 1 09:00",
    "timezone": "UTC",
    "emailChannel": "email",
    "policy": "ProductDigestPolicy",
    "renderer": "ProductDigestRenderer"
};
```

`ReceiptAwareEmailAdapter` is your application's durable email adapter. Legacy digest storage also uses `storage.digests`. See [Named Email Digests](../guides/email-digests.md#configuring-a-digest) for the existing renderer and subscription contracts, or the [Upgrade Guide](../upgrade-guide.md#upgrading-to-1.3.0) to migrate.
