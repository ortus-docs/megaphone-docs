# EventMetadata

### init

Creates an `EventMetadata` instance for the given event type and payload.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/metadata/using](https://api.slack.com/metadata/using)
{% endhint %}

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>type</td><td><code>string</code></td><td>true</td><td></td><td>The event type to track metadata for.</td></tr><tr><td>payload</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>The metadata to track.</td></tr></tbody></table>

**Return**: The `EventMetadata` instance.

### getMemento

Converts the `EventMetadata` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `EventMetadata` instance.

```js
{
    "event_type": "type",
    "event_payload": { ... } 
}
```
