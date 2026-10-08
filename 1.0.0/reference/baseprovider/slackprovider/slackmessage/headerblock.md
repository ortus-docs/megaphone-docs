# HeaderBlock

### init

Creates an `HeaderBlock` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#header](https://api.slack.com/reference/block-kit/blocks#header)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text of the header. Maximum length of 150 characters.</td></tr><tr><td>callback</td><td><code>function</code></td><td>false</td><td><code>null</code></td><td>An optional callback to configure the new <code>HeaderBlock</code>.</td></tr></tbody></table>

**Return**: The `HeaderBlock` instance.

### id

Sets a custom identifier for the `HeaderBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>HeaderBlock</code> instance.</td></tr></tbody></table>

**Return**: The `HeaderBlock` instance.

### getMemento

Converts the `HeaderBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `HeaderBlock` instance.

```cfscript
{
    "type": "header",
    "text": {
        "type": "plain_text",
        "text": "My Header"
    },
    "block_id": "optional"
}
```
