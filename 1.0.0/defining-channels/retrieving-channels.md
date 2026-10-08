# Retrieving Channels

Though not a common use case, you can retrieve configured Channel instances using the Megaphone Channel WireBox DSL.

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
                        "table": "megaphone_notifications",
                        "datasource": "megaphone"
                    }
                }
            }
        };
    }

}
</code></pre>

```cfscript
var databaseChannel = wirebox.getInstance( "megaphone:database" );
```
