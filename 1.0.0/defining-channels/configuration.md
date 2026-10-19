# Configuration

Like cbmailservices and cbfs, Megaphone creates channels to send `Notification` instances on.  Channels are created by defining `provider` and any necessary `properties`.

<pre class="language-js"><code class="lang-js"><strong>// config/modules/megaphone.cfc
</strong>component {

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
</code></pre>

Each Provider defines the required and optional `properties` it accepts.  Refer to the individual Provider documentation for more information.

### Named email digests

[EmailDigestProvider](../providers/emaildigestprovider.md) is a collection channel. Configure its named definitions under `digests` and its persistence settings under `storage.digests`. Each definition selects an ordinary receipt-aware email channel for its eventual summary. See [Named Email Digests](../guides/email-digests.md#configuring-a-digest) for the complete configuration.

### Digests

Megaphone 1.3.0 adds [DigestProvider](../providers/digestprovider.md) for provider-independent collection. Register named channels with `properties.digestName`, and configure each digest with a policy, schedule, and summary Notification mapping. The summary chooses its own delivery channels. Storage remains under `storage.digests`. See [Digests](../guides/digests.md#configuring-a-digest) for daily and weekly examples.
