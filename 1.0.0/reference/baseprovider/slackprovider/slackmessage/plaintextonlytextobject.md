# PlainTextOnlyTextObject

### init

Creates a `PlainTextOnlyTextObject` instance.

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text to display.</td></tr><tr><td>maxLength</td><td><code>numeric</code></td><td>false</td><td><code>3000</code></td><td>The maximum length for the text object. Text over the maximum length will be truncated at the maximum length with <code>...</code> added at the end.</td></tr><tr><td>minLength</td><td><code>numeric</code></td><td>false</td><td><code>1</code></td><td>The minimum length for the text object.</td></tr></tbody></table>

**Return**: The `PlainTextOnlyTextObject` instance.

### emoji

Sets the `PlainTextOnlyTextObject` to interpret emoji in the text field.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `PlainTextOnlyTextObject` instance.

### getMemento

Converts the `PlainTextOnlyTextObject` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `PlainTextOnlyTextObject` instance.

```js
{
    "type": "plain_text",
    "text": "Some text here :tada:",
    "emoji": true
}
```
