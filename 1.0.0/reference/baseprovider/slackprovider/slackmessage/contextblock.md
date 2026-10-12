# ContextBlock

### init

Creates an `ContextBlock` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#context](https://api.slack.com/reference/block-kit/blocks#context)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `ContextBlock` instance.

### id

Sets a custom identifier for the `ContextBlock` instance.

<table><thead><tr><th width="151">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>ActionsBlock</code> instance.</td></tr></tbody></table>

**Return**: The `ContextBlock` instance.

### image

Adds an `ImageElement` to this `ContextBlock` instance.

{% content-ref url="imageelement.md" %}
[imageelement.md](imageelement.md)
{% endcontent-ref %}

<table><thead><tr><th width="164">Name</th><th width="149">Type</th><th width="94">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>imageUrl</td><td><code>string</code></td><td>true</td><td></td><td>The url for the image.</td></tr><tr><td>altText</td><td><code>string</code></td><td>false</td><td><code>""</code> (empty string)</td><td>The alt text for the image.</td></tr></tbody></table>

**Return**: The new `ImageElement` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException` when more than 25 elements are added.
{% endhint %}

### text

Adds a `TextObject` to this `ContextBlock` instance.

{% content-ref url="textobject.md" %}
[textobject.md](textobject.md)
{% endcontent-ref %}

<table><thead><tr><th width="164">Name</th><th width="149">Type</th><th width="94">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the <code>TextObject</code>.</td></tr></tbody></table>

**Return**: The new `TextObject` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException` when more than 25 elements are added.
{% endhint %}

### getMemento

Converts the `ContextBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ContextBlock` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException` when either 0 or more than 25 elements are present.
{% endhint %}

```js
{
    "type": "context"
    "elements": [ ... ],
    "block_id": "optional"
}
```
