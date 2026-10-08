# SlackMessage

### to

Sets the channel, private group, or IM channel to send the message to.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/methods/chat.postMessage#arg\_channel](https://api.slack.com/methods/chat.postMessage#arg_channel)
{% endhint %}

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>channel</td><td><code>string</code></td><td>true</td><td></td><td>The channel, private group, or IM channel to send the message to.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### text

Sets the text of the message.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/methods/chat.postMessage#arg\_text](https://api.slack.com/methods/chat.postMessage#arg_text)
{% endhint %}

<table><thead><tr><th width="139">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text of the message.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### actionsBlock

Creates and adds an [Actions Block](https://api.slack.com/reference/block-kit/blocks#actions) to the message's `blocks`. After creating the block, it will call the provided `callback` function to configure the Actions Block.

<table><thead><tr><th width="131">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>callback</td><td><code>function</code></td><td>true</td><td></td><td>The callback function to configure the created Actions Block.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="actionsblock.md" %}
[actionsblock.md](actionsblock.md)
{% endcontent-ref %}

### contextBlock

Creates and adds a [Context Block](https://api.slack.com/reference/block-kit/blocks#context) to the message's `blocks`. After creating the block, it will call the provided `callback` function to configure the Context Block.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>callback</td><td><code>function</code></td><td>true</td><td></td><td>The callback function to configure the created Context Block.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="contextblock.md" %}
[contextblock.md](contextblock.md)
{% endcontent-ref %}

### dividerBlock

Creates and adds a [Divider Block](https://api.slack.com/reference/block-kit/blocks#divider) to the message's `blocks`.

<table><thead><tr><th width="176">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="dividerblock.md" %}
[dividerblock.md](dividerblock.md)
{% endcontent-ref %}

### headerBlock

Creates and adds a [Header Block](https://api.slack.com/reference/block-kit/blocks#header) to the message's `blocks` with the passed-in `text`. If a `callback` function is provided, it is called to configure the Header Block.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text for the header.</td></tr><tr><td>callback</td><td><code>function</code></td><td>false</td><td><code>null</code></td><td>The callback function to configure the created Header Block.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="headerblock.md" %}
[headerblock.md](headerblock.md)
{% endcontent-ref %}

### imageBlock

Creates and adds an [Image Block](https://api.slack.com/reference/block-kit/blocks#image) to the message's `blocks` with the passed-in `imageUrl`.

If `altText` is not provided when constructing the Image Block, it must be provided in the `callback`.&#x20;

If a `callback` function is provided, it is called to configure the Image Block.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>imageUrl</td><td><code>string</code></td><td>true</td><td></td><td>The url for the image of the block.</td></tr><tr><td>altText</td><td><code>string</code></td><td>false</td><td><code>""</code> (empty string)</td><td>The alt text for the image.  If no alt text is provided, it must be configured in the <code>callback</code>.</td></tr><tr><td>callback</td><td><code>function</code></td><td>false</td><td><code>null</code></td><td>The callback function to configure the created Image Block.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="imageblock.md" %}
[imageblock.md](imageblock.md)
{% endcontent-ref %}

### sectionBlock

Creates and adds a Section Block to the message's `blocks`.

The provided `callback` function is called to configure the Section Block.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>callback</td><td><code>function</code></td><td>true</td><td></td><td>The callback function to configure the created Section Block.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="sectionblock.md" %}
[sectionblock.md](sectionblock.md)
{% endcontent-ref %}

### emoji

Sets the icon of the Slack Message to be the provided emoji.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>icon</td><td><code>string</code></td><td>true</td><td></td><td>The emoji code to set as the icon, e.g. <code>:chart_with_upwards_trend:</code></td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### image

Sets the icon of the Slack Message to be the provided image URL.

<table><thead><tr><th width="164">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>image</td><td><code>string</code></td><td>true</td><td></td><td>The image URL to set as the icon.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### metadata

Add metadata for the given event type.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>eventType</td><td><code>string</code></td><td>true</td><td></td><td>The event type to track metadata for.</td></tr><tr><td>payload</td><td><code>struct</code></td><td>false</td><td><code>{}</code></td><td>The metadata to track.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

{% content-ref url="eventmetadata.md" %}
[eventmetadata.md](eventmetadata.md)
{% endcontent-ref %}

### disableMarkdownParsing

Disables markdown parsing of the Slack Message text.

<table><thead><tr><th width="131">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### unfurlLinks

Pass `true` to enable unfurling of primarily text-based content.

<table><thead><tr><th width="145">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>unfurlLinks</td><td><code>boolean</code></td><td>false</td><td><code>true</code></td><td>Boolean flag to unfurl links or not.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### unfurlMedia

Pass `true` to enable unfurling of media content.

<table><thead><tr><th width="138">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>unfurlMedia</td><td><code>boolean</code></td><td>false</td><td><code>true</code></td><td>Boolean flag to unfurl media or not.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### username

Sets the username for the Slack Bot.

<table><thead><tr><th width="138">Name</th><th width="151">Type</th><th width="113">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>username</td><td><code>string</code></td><td>true</td><td></td><td>The username to use for the Slack Bot.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.

### getMemento

Returns a serializable representation of this Slack Message.

<table><thead><tr><th width="176">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) The serializable Slack Message

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException`
{% endhint %}

### dump

Sends either the memento or a link to the Slack's [BlockKit Builder](https://app.slack.com/block-kit-builder/) to `writeDump`.

<table><thead><tr><th width="176">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>raw</td><td><code>boolean</code></td><td>false</td><td><code>false</code></td><td>If true, outputs the memento of the message.  Otherwise, outputs a URL to Slack's <a href="https://app.slack.com/block-kit-builder/TBHQJRCEN">BlockKit Builder</a>.</td></tr></tbody></table>

{% hint style="info" %}
This method also accepts all the same arguments as [`writeDump`](https://cfdocs.org/writedump).
{% endhint %}

**Return**: (`void`)&#x20;

### when

Control-flow helper to have `if` / `else` statements with method chaining.

<table><thead><tr><th width="176">Name</th><th width="151">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>target</td><td><code>boolean</code></td><td>true</td><td></td><td>The boolean evaluator.</td></tr><tr><td>success</td><td><code>function</code></td><td>true</td><td></td><td>The callback to execute if the boolean value is <code>true</code>.</td></tr><tr><td>failure</td><td><code>function</code></td><td>false</td><td><code>null</code></td><td>The optional callback to execute if the boolean value is <code>false</code>.</td></tr></tbody></table>

**Return**: The `SlackMessage` instance.
