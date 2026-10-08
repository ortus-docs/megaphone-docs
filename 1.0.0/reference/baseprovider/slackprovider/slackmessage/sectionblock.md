# SectionBlock

### init

Creates an `SectionBlock` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#section](https://api.slack.com/reference/block-kit/blocks#section)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `SectionBlock` instance.

### id

Sets a custom identifier for the `SectionBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>SectionBlock</code> instance.</td></tr></tbody></table>

**Return**: The `SectionBlock` instance.

### text

Sets the text for the `SectionBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the section. Maximum of 3000 characters.</td></tr></tbody></table>

**Return**: The `SectionBlock` instance.

### field

Adds a text field to the `SectionBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the new text field. Maximum of 2000 characters.</td></tr></tbody></table>

**Return**: The `TextObject` added to the `SectionBlock` instance.

### accessory

Sets the accessory for the `SectionBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>element</td><td><code>BlockKitElement</code></td><td>true</td><td></td><td>The Slack BlockKit element to set as the accessory for this <code>SectionBlock</code> instance.</td></tr></tbody></table>

**Return**: The `SectionBlock` instance.

### getMemento

Converts the `SectionBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `SectionBlock` instance.

```cfscript
{
    "type": "section",
    "text": {
        "type": "text",
        "text": "optional, either this or fields must be set"
    },
    "fields": [ ... ],
    "accessory": { ... },
    "block_id": "optional"
}
```
