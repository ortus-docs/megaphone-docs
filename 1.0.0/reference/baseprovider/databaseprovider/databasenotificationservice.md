# DatabaseNotificationService

### getNotifications

Returns all database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getReadNotifications

Returns all read database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve read notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve read notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of read notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}

### getUnreadNotifications

Returns all unread database notifications for a Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>notifiable</td><td><a href="../../inotifiable.md"><code>INotifiable</code></a></td><td>true</td><td></td><td>The <a href="../../inotifiable.md"><code>INotifiable</code></a> instance to retrieve unread notifications for.</td></tr><tr><td>channelName</td><td><code>string</code></td><td>false</td><td><code>database</code></td><td>The name of the channel to retrieve unread notifications from.  Defaults to <code>database</code>.</td></tr><tr><td>initialPage</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The initial page of the returned <code>DatabaseNotificationCursor</code>.</td></tr><tr><td>maxRows</td><td><code>numeric</code></td><td>false</td><td><code>25</code></td><td>The number of rows per page for the returned <code>DatabaseNotificationCursor</code>.</td></tr></tbody></table>

**Return**: A `DatabaseNotificationCursor` to interact with a paginated list of unread notifications for the notifiable.

{% hint style="danger" %}
**Throws:** `Megaphone.Configuration.MissingChannel`

**Throws:** `Megaphone.Configuration.InvalidChannelProvider`
{% endhint %}
