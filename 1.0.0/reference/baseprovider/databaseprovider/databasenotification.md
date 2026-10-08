# DatabaseNotification

### markAsRead

Marks the notification as read and updates the database.

<table><thead><tr><th width="156">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>readDate</td><td><code>date</code></td><td>false</td><td><code>now()</code></td><td>The date to use when marking the notification as read.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### markAsRead

Deletes the notification from the database.

<table><thead><tr><th width="156">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### getMemento

Returns a serializable representation of this `DatabaseNotification`.

```cfscript
{
    "id": getId(),
    "type": getType(),
    "notifiableType": getNotifiableType(),
    "notifiableId": getNotifiableId(),
    "data": getData(),
    "readDate": getReadDate(),
    "createdDate": getCreatedDate()
}
```

<table><thead><tr><th width="156">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) The memento of this `DatabaseNotification` instance.

### populateFromDatabaseRow

Returns the Channel instance the notification belongs to.

This method also handles deserializing the `data` property from JSON.

<table><thead><tr><th width="156">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>properties</td><td><code>struct</code></td><td>true</td><td></td><td>The properties to set for this <code>DatabaseNotification</code> instance.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

## Accessors

### getChannel

Returns the Channel instance the notification belongs to.

<table><thead><tr><th width="191">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`BaseProvider`) The Channel instance.

### getId

Returns the id in the database of this notification.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`string`) The `DatabaseNotification` id.

### getType

Returns the Notification Type of the Notification. Populated in the database from calling [`Notification#getNotificationType`](../../basenotification.md#getnotificationtype) which defaults to the [WireBox id](https://wirebox.ortusbooks.com/usage/injection-dsl/wirebox-namespace#id-2nd-level-dsl) of the notification.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`string`) The Notification Type of the `DatabaseNotification`.

### getNotifiableType

Returns the stored Notifiable type of the Notification. Populated in the database from calling [`INotifiable#getNotifiableType`](../../inotifiable.md#getnotifiabletype) when storing the notification.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`string`) The Notifiable type of the `DatabaseNotification`.

### getNotifiableId

Returns the stored Notifiable id of the Notification. Populated in the database from calling [`INotifiable#getNotifiableId`](../../inotifiable.md#getnotifiableid) when storing the notification.

<table><thead><tr><th width="178">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`string`) The Notifiable id of the `DatabaseNotification`.



### getData

Returns the data sent for the notification. Populated in the database from calling [`toDatabase`](../../../providers/databaseprovider.md#todatabase) on the Notification instance when storing the notification.

<table><thead><tr><th width="174">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) The data of the `DatabaseNotification`.

### getReadDate

Returns the read date of the notification.  Returns an empty string if the notification hasn't been read.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`date`) The read date of the `DatabaseNotification`.

### getCreatedDate

Returns the created date of the notification.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`date`) The created date of the `DatabaseNotification`.

{% hint style="danger" %}
The setters of this component should be considered `private`.
{% endhint %}

### setChannel

Sets the Channel instance the notification belongs to.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channel</td><td><code>BaseProvider</code></td><td>true</td><td></td><td>The Channel instance the notification belongs to.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setId

Sets the id of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The id of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setType

Sets the type of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>type</td><td><code>string</code></td><td>true</td><td></td><td>The type of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setNotifiableType

Sets the Notifiable type of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiableType</td><td><code>string</code></td><td>true</td><td></td><td>The Notifiable type of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setNotifiableId

Sets the Notifiable id of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiableId</td><td><code>string</code></td><td>true</td><td></td><td>The Notifiable id of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setData

Sets the data of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>data</td><td><code>struct</code></td><td>true</td><td></td><td>The data of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setReadDate

Sets the read date of the `DatabaseNotification`.

{% hint style="warning" %}
Consider using the `markAsRead` method instead of interacting with this setter directly as the `markAsRead` method will also update the database.
{% endhint %}

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>readDate</td><td><code>date</code></td><td>true</td><td></td><td>The read date of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.

### setReadDate

Sets the created date of the `DatabaseNotification`.

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>createdDate</td><td><code>date</code></td><td>true</td><td></td><td>The created date of the <code>DatabaseNotification</code>.</td></tr></tbody></table>

**Return**: This `DatabaseNotification` instance.
