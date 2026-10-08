# DatabaseProvider

The `DatabaseProvider` stores notifications in a table that you can then query to show in your application.  It also provides a `DatabaseNotificationService` to interact with the notifications for a `Notifiable` including pagination and marking as read.

### Requirements

To use the `DatabaseProvider`, you need a table to store the notifications in.  A migration is provided in Megaphone that you can copy to your application to use.  If you are not using `cfmigrations`, create a table that has the same structure in your database.  The table name can be customized, if needed.

{% tabs %}
{% tab title="CFMigrations" %}
```cfscript
component {

    function up( schema ) {
        schema.create( "megaphone_notifications",  ( t ) => {
            t.guid( "id" ).primaryKey();
            t.string( "type" ); // notification wirebox id
            t.string( "notifiableId" );
            t.string( "notifiableType" );
            t.longText( "data" ); // serializeJSON of what is returned from `toDatabase`
            t.timestamp( "readDate" ).nullable();
            t.timestamp( "createdDate" ).withCurrent();

            t.index( "type" );
            t.index( "readDate" );
            t.index( name = "idx_megaphone_notifications_notifiable_index", columns = [ "notifiableId", "notifiableType" ] );
        } );
    }

    function down( schema ) {
        schema.dropIfExists( "megaphone_notifications" );
    }

}
```
{% endtab %}

{% tab title="MySQL" %}
```sql
CREATE TABLE ` megaphone_notifications` (
    `id` NCHAR(36) NOT NULL,
    `type` VARCHAR(255) NOT NULL,
    `notifiableId` VARCHAR(255) NOT NULL,
    `notifiableType` VARCHAR(255) NOT NULL,
    `data` LONGTEXT NOT NULL,
    `readDate` TIMESTAMP NULL DEFAULT NULL,
    `createdDate` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT `pk_megaphone_notifications_id` PRIMARY KEY (`id`),
    INDEX `idx_megaphone_notifications_type` (`type`),
    INDEX `idx_megaphone_notifications_readDate` (`readDate`),
    INDEX `idx_megaphone_notifications_notifiable_index` (`notifiableId`, `notifiableType`)
)
```
{% endtab %}

{% tab title="SQL Server" %}
```sql
CREATE TABLE [megaphone_notifications] (
    [id] uniqueidentifier NOT NULL,
    [type] VARCHAR(255) NOT NULL,
    [notifiableId] VARCHAR(255) NOT NULL,
    [notifiableType] VARCHAR(255) NOT NULL,
    [data] VARCHAR(MAX) NOT NULL,
    [readDate] DATETIME2,
    [createdDate] DATETIME2 NOT NULL CONSTRAINT [df_megaphone_notifications_createdDate] DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT [pk_megaphone_notifications_id] PRIMARY KEY ([id]),
    INDEX [idx_megaphone_notifications_type] ([type]),
    INDEX [idx_megaphone_notifications_readDate] ([readDate]),
    INDEX [idx_megaphone_notifications_notifiable_index] ([notifiableId], [notifiableType])
)
```
{% endtab %}
{% endtabs %}

### Configuration

The `DatabaseProvider` accepts the following properties:

```json
{
    "tableName": "megaphone_notifications",
    "datasource": null,
    "queryOptions": {}
}
```

These can be different for each channel you configure using the `DatabaseProvider`.

```cfscript
moduleSettings = {
    "megaphone": {
        "channels": {
            "db1": {
                "provider": "DatabaseProvider@megaphone",
                "properties": { "datasource": "db1" }
            },
            "db2": {
                "provider": "DatabaseProvider@megaphone",
                "properties": { "datasource": "db2" }
            }
        }
    }
};
```

{% hint style="warning" %}
The `datasource` property will override any `datasource` property on the passed-in `queryOptions`.
{% endhint %}

### toDatabase

The `toDatabase` method returns a `struct` of data to save as the body of the notification in the database.  This data will be available when reading the notifications back from the database later.

```cfscript
public struct function toDatabase( required any notifiable ) {
    return {
        "stockSymbol": getStockSymbol(),
        "completionTimestamp": getCompletionTimestamp()
    };
}
```

### Interacting with Database Notifications

Where most Megaphone Providers opearte as fire-and-forget, the notifications sent by the `DatabaseProvider` need to be shown in your application to be of any use.  The `DatabaseProvider` provides a few extra components to help you do this.

Just like sending notifications, there are two ways to retrieve notifications sent through the `DatabaseProvider`: using the `DatabaseNotificationService` and using the `HasDatabaseNotifications` delegate.  These two options return the same results, so use whichever one you prefer.

#### DatabaseNotificationService

The `DatabaseNotificationService` can be injected into any component in your application to retrieve `DatabaseNotification` instances for a `Notifiable`.

```cfscript
component {
    
    property
        name="databaseNotificationService"
        inject="DatabaseNotificationService@megaphone";

    function index( event, rc, prc ) {
        // ...
        var cursor = variables.databaseNotificationService.getUnreadNotifications(
            notifiable = auth().user(),
            channel = "database" // default is "database",
            initialPage = 1 // default is 1,
            maxRows = 25 // default is 25
        );
       // ...
    }
    
}
```

This will return a `DatabaseNotificationCursor` paging over all unread notifications for the passed in `Notifiable`.&#x20;

{% hint style="info" %}
The `DatabaseNotificationService` also includes methods for retrieving read notifications or all notifications.  See the [`DatabaseNotificationService` reference docs](../reference/baseprovider/databaseprovider/databasenotificationservice.md) for more information.
{% endhint %}

{% content-ref url="../reference/baseprovider/databaseprovider/databasenotificationservice.md" %}
[databasenotificationservice.md](../reference/baseprovider/databaseprovider/databasenotificationservice.md)
{% endcontent-ref %}

#### HasDatabaseNotifications

The `HasDatabaseNotifications` delegate allows you to get the notifications directly from a `Notifiable` instance.

```cfscript
component name="User" delegates="HasDatabaseNotifications@megaphone" accessors="true" {

    property name="id";

    public string function getNotifiableId() {
        return getId();
    }

    public string function getNotifiableType() {
        return "User";
    }

}
```

```cfscript
component {

    function index( event, rc, prc ) {
        // ...
        var cursor = auth().user().getUnreadNotifications(
            channel = "database" // default is "database",
            initialPage = 1 // default is 1,
            maxRows = 25 // default is 25
        );
        
        // if you want all the defaults:
        var cursor = auth().user().getUnreadNotifications();
        // ...
    }

}
```

{% content-ref url="../reference/baseprovider/databaseprovider/hasdatabasenotifications.md" %}
[hasdatabasenotifications.md](../reference/baseprovider/databaseprovider/hasdatabasenotifications.md)
{% endcontent-ref %}

#### DatabaseNotificationCursor

The `DatabaseNotificationCursor` provides a way to paginate through the notifications while also being able to either `markAllAsRead` or `deleteAll` of the notifications contained in the cursor.

```cfc
cursor.getPagination(); // { "maxRows": 25, "totalPages": 1, "offset": 0, "page": 1, "totalRecords": 5 }
cursor.getResults(); // [ DatabaseNotification ]
for ( var notification in cursor.getResults() ) {
    notification.getMemento(); // { id, type, notifiableType, notifiableId, data, readDate, createdDate }
    notification.getData(); // struct / already deserialized
    notification.getType(); // string — notification wirebox id
    notification.markAsRead( readDate = now() ); // sets and saves the readDate, default = now()
    notification.delete(); // deletes the notification from the database
}
cursor.hasPrevious(); // boolean
cursor.previous(); // loads previous page from database
cursor.hasNext(); // boolean
cursor.next(); // loads next page from database
cursor.markAllAsRead( readDate = now() ); // marks all as read, not just current page. default = now()
cursor.deleteAll(); // deletes all, not just current page
```

{% content-ref url="../reference/baseprovider/databaseprovider/databasenotificationcursor.md" %}
[databasenotificationcursor.md](../reference/baseprovider/databaseprovider/databasenotificationcursor.md)
{% endcontent-ref %}

#### DatabaseNotification

The component returned as the notification inside the `DatabaseNotificationCursor` is an instance of `DatabaseNotification`. This allows you to retrieve the data you sent as well as interact with the `DatabaseNotification` by checking the sending `Notification` type, marking the `DatabaseNotification` as read or deleting the `DatabaseNotification`.

```cfscript
notification.getMemento(); // { id, type, notifiableType, notifiableId, data, readDate, createdDate }
notification.getData(); // struct / already deserialized
notification.getType(); // string — notification wirebox id
notification.markAsRead( readDate = now() ); // sets and saves the readDate, default = now()
notification.delete(); // deletes the notification from the database
```

{% content-ref url="../reference/baseprovider/databaseprovider/databasenotification.md" %}
[databasenotification.md](../reference/baseprovider/databaseprovider/databasenotification.md)
{% endcontent-ref %}
