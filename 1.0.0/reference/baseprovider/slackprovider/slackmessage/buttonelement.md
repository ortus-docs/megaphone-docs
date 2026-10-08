# ButtonElement

### init

Creates a `ButtonElement` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/block-elements#button](https://api.slack.com/reference/block-kit/block-elements#button)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the button. Maximum length of 75.</td></tr><tr><td>callback</td><td><code>function</code></td><td>false</td><td><code>null</code></td><td>A callback function to configure the <code>ButtonElement</code>.</td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### url

Sets the link URL for the `ButtonElement` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>linkUrl</td><td><code>string</code></td><td>true</td><td></td><td>The link URL for the button instance. Maximum length of 3000 characters.</td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### id

Sets a custom identifier for the `ButtonElement` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>ButtonElement</code> instance. Maximum of 255 characters.</td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### value

Sets the value for the `ButtonElement` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>value</td><td><code>string</code></td><td>true</td><td></td><td>The value for the <code>ButtonElement</code> instance.  Maximum of 2000 characters.</td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### primary

Sets the style of the `ButtonElement` to `primary`.

<table><thead><tr><th width="173">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### danger

Sets the style of the `ButtonElement` to `danger`.

<table><thead><tr><th width="173">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### confirm

Sets the confirm object for the `ButtonElement` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text of the confirm object.</td></tr><tr><td>callback</td><td><code>function</code></td><td>false</td><td></td><td>A callback function to configure the confirm object.</td></tr></tbody></table>

**Return**: The newly created `ConfirmObject` instance.

### accessibilityLabel

Sets the accessibility label for the button.

<table><thead><tr><th width="102">Name</th><th width="128">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>label</td><td><code>string</code></td><td>true</td><td></td><td>The accessibility label for the button.  Maximum length of 75 characters.</td></tr></tbody></table>

**Return**: The `ButtonElement` instance.

### getMemento

Converts the `ButtonElement` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ButtonElement` instance.

```cfscript
{
    "type": "button",
    "text": {
        "type": "plain_text",
        "text": "Click me"
    },
    "action_id": "button_click-me",
    "style": "optional, primary or danger",
    "confirm": {
        "title": {
            "type": "plain_text",
            "text": "Are you sure?"
        },
        "text": {
            "type": "plain_text",
            "text": "This will do some thing."
        },
        "confirm": {
            "type": "plain_text",
            "text": "Yes"
        },
        "deny": {
            "type": "plain_text",
            "text": "No"
        }
    },
    "accessibility_label": "optional, Button to do some thing"
}
```
