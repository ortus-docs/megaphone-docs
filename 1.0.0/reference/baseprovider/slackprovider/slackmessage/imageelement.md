# ImageElement

### init

Creates an `ImageElement` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/block-elements#image](https://api.slack.com/reference/block-kit/block-elements#image)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>imageUrl</td><td><code>string</code></td><td>true</td><td></td><td>The url for the image.</td></tr><tr><td>altText</td><td><code>string</code></td><td>false</td><td><code>""</code> (empty string)</td><td>The alt text for the image.  If alt text is not provided in the constructor, it must be provided using the <code>alt</code> method before serializing.</td></tr></tbody></table>

**Return**: The `ImageElement` instance.

### alt

Sets the alt text for the `ImageElement` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>altText</td><td><code>string</code></td><td>true</td><td></td><td>The alt text for the <code>ImageElement</code>.</td></tr></tbody></table>

**Return**: The `ImageElement` instance.

### getMemento

Converts the `ImageElement` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ImageElement` instance.

```js
{
    "type": "image",
    "image_url": "https://placekitten.com/200/300",
    "alt_text": "A cute kitten"
}
```
