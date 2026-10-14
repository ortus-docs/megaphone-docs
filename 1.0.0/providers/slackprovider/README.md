# SlackProvider

The `SlackProvider` sends notifications to Slack using [Hyper](https://forgebox.io/view/hyper).

### Requirements

To use the `SlackProvider`, you need Hyper installed. This is **not** installed by Megaphone.  If you do not have Hyper installed, an exception will be thrown if you try to define a channel with the `SlackProvider`.

Additionally, the `SlackProvider` requires a `token` to be set in the configuration.  You will need a [Slack App installed](https://api.slack.com/apps?new_app=1) in your Slack Workspace to retrieve that token. The Slack App will need the `chat:write`, `chat:write.public`, and `chat:write.customize` scopes, at a minimum.  It may need more scopes if you want to send messages to private channels or direct messages, for example. Consult the [Slack App permission documentation](https://api.slack.com/scopes) to determine what scopes your application needs.

### Configuration

The `SlackProvider` accepts the following properties:

```json
{
    "token": null,
    "defaultChannel": "##general"
}
```

### toSlack

The `toSlack` method should return a `SlackMessage` instance.

```js
public struct function toSlack( notifiable, newSlackMessage ) {
    return newSlackMessage()
        .to( "##payments" )
        .text( "One of your invoices has been paid!" )
        .headerBlock( "Invoice Paid" )
        .contextBlock( ( block ) => {
            block.text( "Customer ###notifiable.getCustomerId()#" );
        } )
        .sectionBlock( ( block ) => {
            block.text( "An invoice has been paid." );
            block.field( "*Invoice No:*#chr( 10 )##getInvoiceNumber()#" ).markdown();
            block.field( "*Invoice Recipient:*#chr( 10 )##notifiable.getEmail()#" ).markdown();
        } )
        .dividerBlock()
        .sectionBlock( ( block ) => {
            block.text( "Congratulations!" );
        } );
}
```

Megaphone includes components to build out a Slack BlockKit payload using a fluent API. This is the preferred way to build a Slack Message.

{% content-ref url="slack-blockkit.md" %}
[slack-blockkit.md](slack-blockkit.md)
{% endcontent-ref %}

Additionally, you can return a struct representation of the Slack BlockKit payload instead of using `SlackMessage` or the included `SlackBlockKit` components.

```js
public struct function toSlack( notifiable, newSlackMessage ) {
    return {
        "channel": "##general",
        "text": "A super simple message"
    };
}
```

### routeNotificationForSlack

You can let the `Notifiable` instance define the channel to send the Slack Notification by adding a `routeNotificationForSlack` method.

```js
component accessors="true" {
    
    property name="slackDirectMessageID";
    
    public string function routeNotificationForSlack() {
        return getSlackDirectMessageID();
    }

}
```

Additionally, you may need to use a different token for a notification.  Reasons for this can include sending your notification to multiple different Slack Workspaces depending on the `Notifiable` instance's configuration.  In these cases, you need to return a `SlackRoute` instance from this method.  A helper function is provided to create the `SlackRoute` as a parameter to the `routeNotificationForSlack` method.

```js
component accessors="true" {
    
    property name="slackDirectMessageID";
    property name="slackToken";
    
    public SlackRoute function routeNotificationForSlack( newSlackRoute ) {
        return newSlackRoute(
            getSlackDirectMessageID(),
            getSlackToken()
        );
    }

}
```
