# DatabaseProvider

See the `BaseProvider` for inherited methods.

{% content-ref url="../" %}
[..](../)
{% endcontent-ref %}

### getTableName

Returns the configured table name for this Channel.

The default is `megaphone_notifications`. It can be overridden by setting the `table` property.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (string) The name of the table to use for storing and retrieving notifications.

### getQueryOptions

Returns the configured query options for this Provider.  This is eventually passed to `queryExecute`.

The default is an empty struct (`{}`). It can be overridden by setting the `queryOptions` property.

If the `datasource` property is set, it will be set on the `queryOptions` returned from this method.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The struct of query options for this Channel.

