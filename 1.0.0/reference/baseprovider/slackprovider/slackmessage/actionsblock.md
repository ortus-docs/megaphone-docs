# ActionsBlock

### init

Creates an `ActionsBlock` instance for the given event type and payload.

{% hint style="info" %}
Slack Docs: [https://api.slack.com/reference/block-kit/blocks#actions](https://api.slack.com/reference/block-kit/blocks#actions)
{% endhint %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: The `ActionsBlock` instance.

### id

Sets a custom identifier for the `ActionsBlock` instance.

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>id</td><td><code>string</code></td><td>true</td><td></td><td>The custom identifier for the <code>ActionsBlock</code> instance.</td></tr></tbody></table>

**Return**: The `ActionsBlock` instance.

### button

Adds a `ButtonElement` to this `ActionsBlock` instance.

{% content-ref url="buttonelement.md" %}
[buttonelement.md](buttonelement.md)
{% endcontent-ref %}

<table><thead><tr><th width="164">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>text</td><td><code>string</code></td><td>true</td><td></td><td>The text of the new button element.</td></tr></tbody></table>

**Return**: The new `ButtonElement` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException` when more than 25 elements are added.
{% endhint %}

### getMemento

Converts the `ActionsBlock` instance to a serializable format.

<table><thead><tr><th width="181">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (`struct`) A struct representing this `ActionsBlock` instance.

{% hint style="danger" %}
**Throws:** `Megaphone.Provider.SlackBlockException` when either 0 or more than 25 elements are present.
{% endhint %}

```cfscript
{
    "type": "actions"
    "elements": [ ... ],
    "block_id": "optional"
}
```
