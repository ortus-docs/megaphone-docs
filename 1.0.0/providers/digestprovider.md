# DigestProvider

`DigestProvider` collects items for a named digest. The digest's schedule determines when Megaphone creates a summary Notification. That Notification chooses the delivery channels and provides their content.

This Provider requires Megaphone 1.3.0. See [Digests](../guides/digests.md) for a complete daily/weekly audition example.

### Configuration

Each collection channel requires `properties.digestName`:

```cfml
"dailyAuditions": {
    "provider": "DigestProvider@megaphone",
    "properties": { "digestName": "dailyAuditions" }
}
```

Configure the named definition under `moduleSettings.megaphone.digests`:

```cfml
"dailyAuditions": {
    "schedule": "everyDayAt 09:00",
    "policy": "AuditionDigestPolicy",
    "notification": "DailyAuditionsNotification"
}
```

The definition supplies a policy and summary Notification mapping. It does not select a delivery channel or a renderer. Its schedule uses the existing native ColdBox integration. An immediate-only definition can omit the schedule.

### toDigest

Return an item from the source Notification's `toDigest()` method:

```cfml
public struct function toDigest( required any notifiable ) {
    return {
        "itemKey": variables.digests.itemKey( [ "audition.published", variables.publicationEventId, 1 ] ),
        "occurredAt": variables.digests.occurredAt( variables.publishedAt ),
        "data": { "auditionId": variables.auditionId }
    };
}
```

`variables.digests` is an injected `DigestService@megaphone`. The key identifies the saved event, `occurredAt` is its original UTC epoch-millisecond timestamp, and `data` contains JSON-safe values. Reusing an item key with different content or an occurrence time fails.

`via()` returns configured channel names such as `[ "dailyAuditions", "weeklyAuditions" ]`. Both channels call `toDigest()` because both use `DigestProvider`. The channel name does not produce a `toDailyAuditions()` or `toWeeklyAuditions()` method. Configuration supplies the digest name; an item cannot override it.

### Subscriptions and delivery

Use [DigestService](../reference/digestservice.md) to activate subscriptions. A missing or stopped subscription suppresses collection; routing does not opt the recipient in.

Use `notify()`, `queue()`, or `queuePlan()` as usual. `plan()` captures collection intents without writing or sending. At the cutoff, the summary Notification receives `batchId`, `digestName`, `cutoffAt`, and selected `items`, then follows normal durable routing. An immediate subscription creates a one-item summary through that same path. No second source delivery route is required.

Returning both ordinary and collection channels requests two delivery products. Summary Notifications themselves cannot select a collection channel. See [Offering Immediate Notifications](../guides/digests.md#offering-immediate-notifications) for the distinction.
