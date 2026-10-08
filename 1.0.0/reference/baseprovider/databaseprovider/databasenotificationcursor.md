# DatabaseNotificationCursor

### configureQuery

Calls the provided callback to configure the underlying query for this `DatabaseNotificationCursor` instance.&#x20;

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>callback</td><td><code>function</code></td><td>true</td><td></td><td>The callback called to configure the underlying query of this <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.

### fetch

Executes the configured query and stores the pagination and results internally. Maps the resulting rows into `DatabaseNotification` instances.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.

### hasNext

Returns `true` if there are more pages left in the cursor.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: `boolean`

### next

Advances the page of the cursor and fetches the results.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.DatabaseNotificationCursor.MaximumPageReached`
{% endhint %}

### hasPrevious

Returns `true` if there are more pages before the current page in the cursor.

<table><thead><tr><th width="157">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: `boolean`

### previous

Reverses the page of the cursor and fetches the results.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.DatabaseNotificationCursor.MinimumPageReached`
{% endhint %}

### markAllAsRead

Marks all the notifications under the cursor as read.

{% hint style="info" %}
This marks all the notifications the cursor applies to, not just the currently loaded page.
{% endhint %}

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>readDate</td><td><code>date</code></td><td>false</td><td><code>now()</code></td><td>The date to use when marking the notifications as read.</td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.

### deleteAll

Deletes all the notifications under the cursor.

{% hint style="info" %}
This deletes all the notifications the cursor applies to, not just the currently loaded page.
{% endhint %}

<table><thead><tr><th width="180">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td><h3></h3></td><td></td><td><h3></h3></td><td></td></tr></tbody></table>

**Return**: The `DatabaseNotificationCursor` instance.
