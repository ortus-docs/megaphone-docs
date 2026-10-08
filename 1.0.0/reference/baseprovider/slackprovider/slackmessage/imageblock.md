# ImageBlock

### init

Creates an `ImageBlock` instance.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#image](https://api.slack.com/reference/block-kit/blocks#image)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>imageUrl</td><td><code>string</code></td><td>true</td><td></td><td>The url for the image.  Maximum of 3000 characters.</td></tr><tr><td>altText</td><td><code>string</code></td><td>false</td><td><code>""</code> (empty string)</td><td>The alt text for the <code>ImageBlock</code>.  If no alt text is provided here, it must be configured using the <code>alt</code> method before serializing.</td></tr></tbody></table>

**Return**: The `ImageBlock` instance.

### id

Sets a custom identifier for the `ImageBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>ImageBlock</code> instance.</td></tr></tbody></table>

**Return**: The `ImageBlock` instance.

### alt

Sets the alt text for the `ImageBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>altText</td><td><code>string</code></td><td>true</td><td></td><td>The alt text for the image. Maximum of 2000 characters.</td></tr></tbody></table>

**Return**: The `ImageBlock` instance.

### title

Sets the title for the `ImageBlock` instance.

<table><thead><tr><th width="104">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>title</td><td><code>string</code></td><td>true</td><td></td><td>The title for the image. Maximum of 2000 characters.</td></tr></tbody></table>

**Return**: The `ImageBlock` instance.

### getMemento

Converts the `ImageBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ImageBlock` instance.

```cfscript
{
    "type": "image",
    "image_url": "https://placekitten.com/200/300",
    "alt_text": "A cute kitten",
    "block_id": "optional"
    "text": {
        "type": "plain_text",
        "text": "My Title"
    },
}
```
