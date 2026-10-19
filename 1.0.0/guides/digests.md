# Digests

A digest collects notification items and decides when to send a summary. The summary is another Notification. Its `via()` method chooses the delivery channels, and its `toEmail()`, `toSMS()`, or other Provider methods supply the content.

For example, CommuniArts can offer daily and weekly audition updates. One user might receive the daily summary by email. Another might receive the weekly summary by SMS. A user can also choose both delivery channels without collecting the same items twice.

{% hint style="info" %}
This API requires Megaphone 1.3.0. The [1.2.0 email digest API](email-digests.md) remains supported with its existing subscription and rendering behavior.
{% endhint %}

### Configuring a digest

Register a collection channel for each named digest:

```cfml
moduleSettings.megaphone.channels.dailyAuditions = {
    "provider": "DigestProvider@megaphone",
    "properties": { "digestName": "dailyAuditions" }
};

moduleSettings.megaphone.channels.weeklyAuditions = {
    "provider": "DigestProvider@megaphone",
    "properties": { "digestName": "weeklyAuditions" }
};

moduleSettings.megaphone.digests = {
    "dailyAuditions": {
        "schedule": "everyDayAt 09:00",
        "policy": "AuditionDigestPolicy",
        "notification": "DailyAuditionsNotification"
    },
    "weeklyAuditions": {
        "schedule": "everyWeekOn 1 09:00",
        "policy": "AuditionDigestPolicy",
        "notification": "WeeklyAuditionsNotification"
    }
};
```

Keep your ordinary email, SMS, or other delivery channel registrations. The digest definition does not select one of those channels or supply a renderer. Its `notification` identifies the summary Notification to create at the cutoff. Different definitions can use different Notification classes or share the same class.

Schedules use the native ColdBox scheduler. Definitions accept shortcut strings, configuration closures, or unstarted task templates, along with an optional `timezone`. Subscriptions can override the schedule and timezone. Use `registerDigest()` to add a definition to the existing module scheduler after application startup. See [Native Scheduling](email-digests.md#scheduling-with-coldbox) for the supported schedule forms.

Apply `2026_10_18_120000_add_generic_digest_publications` after the existing digest migration. Keep `storage.digests`, events, and deliveries on the same transactional datasource. The new migration adds saved resume settings and `megaphone_digest_publications`; it preserves existing subscriptions and pending work. If you use custom table names, apply equivalent changes through your application migrations and configure `storage.digests.publicationsTable`. See the [Upgrade Guide](../upgrade-guide.md).

Configure `queue.notifiableResolver` to load the current Notifiable from its saved type and ID, or return null when it no longer exists. Reuse your resolver from [Durable Delivery](durable-delivery.md). Deleted recipients do not receive summaries. External sends happen after publication commits; call synchronous `notify()` outside your application transactions.

### Adding a subscription

Megaphone stores the subscriptions. Your application can read them to display the user's daily or weekly choice without storing another frequency field:

```cfml
var digests = wirebox.getInstance( "DigestService@megaphone" );

digests.addSubscription(
    notifiable = user,
    digestName = "dailyAuditions",
    timezone = "America/Denver"
);
```

A missing or stopped subscription does not collect items. Returning a collection channel from `via()` does not subscribe the user. Megaphone records the suppression, so activating a subscription later does not replay those items.

### Collecting audition updates

`AuditionPublishedNotification` can read the subscriptions to choose its collection channels:

```cfml
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="digests" inject="DigestService@megaphone";
    property name="auditionId";
    property name="publicationEventId";
    property name="publishedAt";

    public array function via( required any notifiable ) {
        var channels = [];
        for ( var name in [ "dailyAuditions", "weeklyAuditions" ] ) {
            var subscription = variables.digests.getSubscription( arguments.notifiable, name );
            if ( !isNull( subscription ) && subscription.mode != "off" ) {
                channels.append( name );
            }
        }
        return channels;
    }

    public struct function toDigest( required any notifiable ) {
        return {
            "itemKey": variables.digests.itemKey( [ "audition.published", variables.publicationEventId, 1 ] ),
            "occurredAt": variables.digests.occurredAt( variables.publishedAt ),
            "data": { "auditionId": variables.auditionId }
        };
    }

}
```

Both named channels use [DigestProvider](../providers/digestprovider.md), so both call `toDigest()`. They do not call `toDailyAuditions()` or `toWeeklyAuditions()`. The channel's `properties.digestName` chooses the collection. The returned item cannot change that name.

`itemKey()` generates a stable key from ordered identity parts. Use the saved publication event ID and occurrence date on every retry. `occurredAt()` converts the date to UTC epoch milliseconds. The collected `data` is serializable content, not a live Notification instance.

Send the Notification using `notify()`, `queue()`, or `queuePlan()`. `plan()` captures the decision without collecting or sending. Multiple source Notification classes can contribute compatible items to the same digest, and one Notification can enter multiple digests.

### Sending the summary

At the cutoff, Megaphone populates the configured summary Notification with `batchId`, `digestName`, `cutoffAt`, and the selected `items`. For example, `DailyAuditionsNotification` can choose its delivery channels from the Notifiable's preferences:

```cfml
component extends="megaphone.models.BaseNotification" accessors="true" {

    property name="batchId";
    property name="digestName";
    property name="cutoffAt";
    property name="items";
    property name="auditionViews" inject="AuditionViews";

    public array function via( required any notifiable ) {
        return arguments.notifiable.getAuditionDeliveryChannels();
    }

    public Mail function toEmail( required any notifiable, required any newMail ) {
        return arguments
            .newMail(
                to = arguments.notifiable.getEmail(),
                from = "updates@example.com",
                subject = "Your daily audition updates",
                type = "html"
            )
            .setBody( variables.auditionViews.dailyEmail( variables.items, arguments.notifiable ) );
    }

    public string function toSMS( required any notifiable ) {
        return variables.auditionViews.dailyText( variables.items, arguments.notifiable );
    }

}
```

`getAuditionDeliveryChannels()` and `AuditionViews` are application-specific examples. Megaphone does not supply an SMS Provider; use the formatter signature and return type required by your chosen Provider. The weekly Notification can use its own subject, grouping, and text. Use the saved `cutoffAt`, in UTC epoch milliseconds, when labeling the summary period so a retry keeps the same period.

If `via()` returns `[ "email", "sms" ]`, Megaphone creates an independent delivery for each channel. A failed SMS can retry without sending an accepted email again. Each delivery Provider must support [Durable Delivery](durable-delivery.md).

Channels are selected once when the summary is published. Current preferences can suppress a pending delivery. Enabling a new channel does not add it to an existing summary. An empty channel selection is saved as a suppressed publication and is not replayed later. Summary Notifications cannot return collection channels.

### Choosing the items

By default, a digest includes every eligible item, newest first, with item-key ordering for ties. Add `selectItems` to the definition to choose or order a subset. Unselected items are permanently discarded at the cutoff. They are not saved for the next digest.

Your policy supplies `canCollect()`, `canSend()`, and `isEligible()`. For example, `AuditionDigestPolicy` can check consent and whether an audition is still visible:

```cfml
component implements="megaphone.models.Interfaces.IDigestPolicy" {

    property name="auditions" inject="AuditionService";

    public boolean function canCollect( required any notifiable, required struct definition, required struct item ) {
        return arguments.notifiable.allowsAuditionNotifications() &&
        isEligible( arguments.notifiable, arguments.definition, arguments.item );
    }

    public boolean function canSend( required any notifiable, required struct definition, required struct subscription ) {
        return arguments.notifiable.allowsAuditionNotifications();
    }

    public boolean function isEligible( required any notifiable, required struct definition, required struct item ) {
        return variables.auditions.isVisibleTo( arguments.item.data.auditionId, arguments.notifiable );
    }

}
```

`allowsAuditionNotifications()` and `AuditionService.isVisibleTo()` are application-specific examples. These checks read current local consent and domain data. An audition that was removed or is no longer visible should not appear in a summary.

Consent, recipient identity, subscription generation, and item eligibility are checked again before transport. If eligibility changes while a Provider prepares its content, preparation is retried with the reduced membership. No replacement items are added to a frozen batch.

### Changing a subscription

Use the lifecycle methods to change a user's choice:

```cfml
// Change timing while keeping eligible pending items.
digests.updateSubscription(
    notifiable = user,
    digestName = "dailyAuditions",
    timezone = "America/Chicago",
    schedule = "everyDayAt 17:00"
);

// Stop collection and suppress deliveries that have not started.
digests.stopSubscription( notifiable = user, digestName = "dailyAuditions" );

// Resume the saved settings for future items.
digests.resumeSubscription( notifiable = user, digestName = "dailyAuditions" );
```

An identical active `addSubscription()` does nothing. Use `updateSubscription()` to change active settings. Updating a stopped subscription changes its saved settings without subscribing the user again. Repeated stop and resume calls are harmless. Updating or resuming a missing subscription raises `Megaphone.Digests.SubscriptionMissing`; stopping one does nothing.

Schedule changes keep eligible pending items and invalidate obsolete unstarted batches. Resuming does not replay suppressed or discarded items. The [DigestService reference](../reference/digestservice.md) covers the method arguments and return values.

### Moving from daily to weekly

Adding a weekly subscription does not stop the daily subscription. Use a transfer when your application offers them as mutually exclusive choices:

```cfml
var result = digests.transferSubscription(
    notifiable = user,
    fromDigest = "dailyAuditions",
    toDigest = "weeklyAuditions",
    pendingItems = "transfer"
);
```

`pendingItems` is required:

| Value | What happens to pending source items |
| --- | --- |
| `transfer` | Eligible unsent items move to the destination. The source retains their admission history. |
| `discard` | Pending source items are suppressed. The destination collects future items. |

The source stops and the destination activates in one transaction. An existing destination keeps its schedule, timezone, and pending items. A stopped destination resumes its saved settings. A missing destination uses its definition's defaults.

For `transfer`, the destination policy must explicitly approve the item format:

```cfml
public boolean function canTransfer(
    required any notifiable,
    required struct sourceDefinition,
    required struct destinationDefinition,
    required array items
) {
    var names = "dailyAuditions,weeklyAuditions";
    return listFindNoCase( names, arguments.sourceDefinition.name ) > 0 &&
        listFindNoCase( names, arguments.destinationDefinition.name ) > 0;
}
```

Add this hook to the policy alongside its normal consent and eligibility methods. It is optional for ordinary collection, but required to accept transferred items. Missing approval, incompatible content, or a policy error aborts the transfer without partial changes. Destination collection and item eligibility checks still apply; ineligible items are suppressed. `discard` does not need format approval.

Matching destination item keys and content deduplicate without replaying delivered, discarded, or suppressed items. Conflicting content fails the whole transfer. A moved source item retains a separate transferred state: an explicit weekly-to-daily transfer can move it back, while a retry of its original publication cannot collect it again.

Once any delivery of a summary has started, that entire batch is excluded from transfer. Megaphone suppresses the remaining unstarted source deliveries and preserves in-flight or uncertain outcomes for reconciliation. It does not republish that batch's items at the destination. For example, if email was accepted while SMS was still pending, the pending SMS is suppressed instead of sending those items again in a weekly summary.

The result contains `source`, `destination`, `transferred`, `deduplicated`, `suppressed`, and `nontransferable`. Repeating a transfer from a stopped source to an active destination does nothing. Transfers to the same digest or from a missing subscription fail.

### Offering immediate notifications

For individual audition notifications, return ordinary delivery channels from `AuditionPublishedNotification.via()` and implement their formatters on that Notification. To offer this alongside the digest choices, start that method's channel array from an application preference method instead of an empty array:

```cfml
var channels = arguments.notifiable.getImmediateAuditionChannels();
```

`getImmediateAuditionChannels()` is an application-specific example. It returns the user's ordinary channels for individual alerts, or an empty array when they do not want individual alerts. The rest of `via()` can still add the user's active digest channels.

When the user switches to individual alerts instead of a digest, stop their old subscriptions explicitly:

```cfml
for ( var name in [ "dailyAuditions", "weeklyAuditions" ] ) {
    digests.stopSubscription( notifiable = user, digestName = name );
}
```

An immediate digest is another choice. Configure an `auditionUpdates` definition with a policy and `notification: "AuditionUpdatesNotification"`, then register a `DigestProvider` channel whose `properties.digestName` is `auditionUpdates`:

```cfml
digests.addSubscription(
    notifiable = user,
    digestName = "auditionUpdates",
    mode = "immediate"
);
```

This creates a one-item summary for each new item using the same policy and summary Notification. An immediate-only definition does not need a schedule, and the preference UI can offer On/Off. The source Notification returns only `[ "auditionUpdates" ]`; it does not need a second ordinary delivery route.

An ordinary immediate Notification uses the source Notification's content. An immediate digest uses the named stream's eligibility, selection, and summary Notification. Returning both ordinary and collection channels deliberately requests both products. For example, an application might send an individual audition alert and also include it in a later weekly summary. Each needs its own preference rule; returning both channels does not make them alternatives.

Delivery channel preferences remain separate from the digest subscription. Switching email to SMS changes how the summary is delivered, not whether it is daily or weekly.
