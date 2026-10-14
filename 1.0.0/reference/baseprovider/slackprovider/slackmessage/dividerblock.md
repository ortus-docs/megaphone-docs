# DividerBlock

### init

Creates an `DividerBlock` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#divider](https://api.slack.com/reference/block-kit/blocks#divider)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `DividerBlock` instance.

### id

Sets a custom identifier for the `DividerBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>DividerBlock</code> instance.</td></tr></tbody></table>

**Return**: The `DividerBlock` instance.

### getMemento

Converts the `DividerBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `DividerBlock` instance.

```js
{
    "type": "divider"
    "block_id": "optional"
}
```
