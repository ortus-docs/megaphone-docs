# Event identities and saved plans

Options are `namespace` (default `megaphone`), `eventKey` (generated if omitted), `version` (1), `occurredAt` and `availableDate` (now), and optional `admitAfter`. Occurrences older than the cutoff return an expired publication without recording work. Equality is admitted.

Persist `plan.getMemento()` when admission must precede worker handoff. Restore it with `new megaphone.models.NotificationPlan({}, []).populateFromMemento(saved)` and call `queuePlan(plan)`. Its version 1 memento contains `event`, `deliveries`, and optional `admitAfter`. It preserves identities, payloads, hashes, destinations, and channel choices. A trusted producer may use `queuePlan(plan, alreadyAdmitted=true)` after validating its original persisted admission. This bypasses the replay cutoff; it does not replan the audience.

Notifications extend `BaseNotification`, implement `via(notifiable)` and the usual `to{Provider}` methods, and may override `getChannelPreferences(notifiable)` to return a channel-name map of `{ enabled, source }`. The default enables the channels returned by `via()`. Synchronous delivery skips disabled channels. Durable delivery records them as suppressed; later opt-in does not revive them.

`getQueueProperties()` defaults to the explicitly populated properties. Notifications configured through setters must override it to return their durable properties. Store JSON data rather than components, closures, credentials, users, or prepared mail/request objects. `getNotificationMapping()` defaults to the WireBox identity; override it when notification type and reconstruction mapping differ. `getDeliveryContext(notifiable, channelName)` provides application routing context, separate from the notification properties.


The event namespace, key, and version are its durable identity. Replay compares the saved payload hash and returns original work. Replanning cannot expand that work. Keep application hashes stable when upgrading existing data.
