# Configuration

Like cbmailservices and cbfs, Megaphone creates channels to send `Notification` instances on.  Channels are created by defining `provider` and any necessary `properties`.

<pre class="language-cfscript"><code class="lang-cfscript"><strong>// config/modules/megaphone.cfc
</strong>component {

    function configure() {
        return {
            "channels": {
                // this is the unique name for the channel
                "database": {
		    "provider": "DatabaseProvider@megaphone",
		    "properties": {
                        // this is the default table name
                        "tableName": "megaphone_notifications",
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
