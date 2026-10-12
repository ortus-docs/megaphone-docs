# DatabaseNotificationKeysetCursor

This cursor is returned by `getNotificationsAfter()`. Use it to load a Notifiable's inbox one page at a time, without numbered pages.

### getResults

Returns the `DatabaseNotification` objects on the current page. Each object retains the ownership and authorization constraints used to load it.

**Return**: An array of `DatabaseNotification` instances.

### getNextCursor

Returns the token for the next page. Pass it as `afterCursor` in a later `getNotificationsAfter()` call.

**Return**: A `string` cursor token.

### hasNext

Checks whether another page is available.

**Return**: A `boolean`.

### next

Loads the next page using the same filters and authorization constraints. It replaces the cursor's current results and returns this cursor.

```js
if ( inbox.hasNext() ) {
    inbox.next();
    var notifications = inbox.getResults();
}
```

**Return**: This `DatabaseNotificationKeysetCursor` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.DatabaseNotificationCursor.MaximumPageReached` if there is no next page. Check `hasNext()` before advancing.
{% endhint %}

### getMemento

Returns a serializable representation of the current page. Notification objects are converted to their mementos:

```js
{
    "results": [ /* Notification mementos */ ],
    "nextCursor": inbox.getNextCursor(),
    "hasMore": inbox.hasNext()
}
```

**Return**: A `struct` containing `results`, `nextCursor`, and `hasMore`.

See [Notification Inboxes](../../../guides/notification-inboxes.md) for request examples and the existing numbered-page cursor APIs.
