# Slack BlockKit

Megaphone provides a fluent BlockKit API to build a `SlackMessage`.  It starts by calling the `newSlackMessage` provided to the `toSlack` method on the Notification.

```cfscript
function toSlack( notifiable, newSlackMessage ) {
    return newSlackMessage();
}
```

For the individual methods and building blocks, please see the reference documentation.

{% content-ref url="../../reference/baseprovider/slackprovider/slackmessage/" %}
[slackmessage](../../reference/baseprovider/slackprovider/slackmessage/)
{% endcontent-ref %}

An important callout is the `dump` method.  This method can `writeDump` a memento of the `SlackMessage` instance when passing the `raw` parameter as `true`.  The default, however, is to `writeDump` a URL that takes you to Slack's [BlockKit Builder](https://app.slack.com/block-kit-builder/) to visually see your message.

```cfscript
public void function dump(
    boolean raw = false,
    string output = "browser",
    string format = "html",
    boolean abort = false,
    string label = "",
    boolean metainfo = false,
    numeric top = 9999,
    string show = "",
    string hide = "",
    numeric keys = 9999,
    boolean expand = true,
    boolean showUDFs = true
);
```

Example URL:

[https://app.slack.com/block-kit-builder/TBHQJRCEN#%7B%22blocks%22:%5B%7B%22text%22:%7B%22text%22:%22Invoice%20Paid%22,%22type%22:%22plain\_text%22%7D,%22type%22:%22header%22%7D,%7B%22elements%22:%5B%7B%22text%22:%22Customer%20%231234%22,%22type%22:%22plain\_text%22%7D%5D,%22type%22:%22context%22%7D,%7B%22fields%22:%5B%7B%22text%22:%22\*Invoice%20No:\*%5Cn1000%22,%22type%22:%22mrkdwn%22%7D,%7B%22text%22:%22\*Invoice%20Recipient:\*%5Cneric@ortussolutions.com%22,%22type%22:%22mrkdwn%22%7D%5D,%22text%22:%7B%22text%22:%22An%20invoice%20has%20been%20paid.%22,%22type%22:%22plain\_text%22%7D,%22type%22:%22section%22%7D,%7B%22type%22:%22divider%22%7D,%7B%22text%22:%7B%22text%22:%22Congratulations!%22,%22type%22:%22plain\_text%22%7D,%22type%22:%22section%22%7D%5D%7D](https://app.slack.com/block-kit-builder/TBHQJRCEN#%7B%22blocks%22:%5B%7B%22text%22:%7B%22text%22:%22Invoice%20Paid%22,%22type%22:%22plain_text%22%7D,%22type%22:%22header%22%7D,%7B%22elements%22:%5B%7B%22text%22:%22Customer%20%231234%22,%22type%22:%22plain_text%22%7D%5D,%22type%22:%22context%22%7D,%7B%22fields%22:%5B%7B%22text%22:%22*Invoice%20No:*%5Cn1000%22,%22type%22:%22mrkdwn%22%7D,%7B%22text%22:%22*Invoice%20Recipient:*%5Cneric@ortussolutions.com%22,%22type%22:%22mrkdwn%22%7D%5D,%22text%22:%7B%22text%22:%22An%20invoice%20has%20been%20paid.%22,%22type%22:%22plain_text%22%7D,%22type%22:%22section%22%7D,%7B%22type%22:%22divider%22%7D,%7B%22text%22:%7B%22text%22:%22Congratulations!%22,%22type%22:%22plain_text%22%7D,%22type%22:%22section%22%7D%5D%7D)
