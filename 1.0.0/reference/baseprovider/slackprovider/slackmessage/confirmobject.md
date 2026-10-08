# ConfirmObject

### init

Creates a `ConfirmObject` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/composition-objects#confirm](https://api.slack.com/reference/block-kit/composition-objects#confirm)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the confirm object.</td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### title

Sets the title for the `ConfirmObject` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>title</td><td><code>string</code></td><td>true</td><td></td><td>The title for the <code>ConfirmObject</code>. Maximum character limit of 100.</td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### text

Sets the text for the `ConfirmObject` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the <code>ConfirmObject</code>. Maximum character limit of 300.</td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### confirm

Sets the confirm button label for the `ConfirmObject` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>label</td><td><code>string</code></td><td>true</td><td></td><td>The confirm button label for the <code>ConfirmObject</code>. Maximum character limit of 30.</td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### deny

Sets the deny button label for the `ConfirmObject` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>label</td><td><code>string</code></td><td>true</td><td></td><td>The deny button label for the <code>ConfirmObject</code>. Maximum character limit of 30.</td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### danger

Marks the `ConfirmObject` as dangerous.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `ConfirmObject` instance.

### getMemento

Converts the `ConfirmObject` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ConfirmObject` instance.

```cfscript
{
    "title": {
        "type": "plain_text",
        "text": "Are you sure?"
    },
    "text": {
        "type": "plain_text",
        "text": "Please confirm this action."
    },
    "confirm": {
        "type": "plain_text",
        "text": "Yes"
    },
    "deny": {
        "type": "plain_text",
        "text": "No"
    },
    "style": "optional, danger"
}
```
