# What's New?

## 1.1.0 (pending release)

Megaphone 1.1.0 adds an inbox, stored preferences, durable delivery, and web push. You can keep using the existing notification and provider APIs.

* [Inboxes](guides/notification-inboxes.md) add recipient access constraints, archive state, read snapshots, history imports, retention, and keyset pages.
* [Preferences](guides/notification-preferences.md) resolve choices by scope and channel. [Push registrations](guides/push-registrations.md) store encrypted subscriptions and exclusions for each notification type.
* [Durable delivery](guides/durable-delivery.md) stores event identities and delivery attempts, fences workers with claim tokens, and keeps uncertain acceptance out of automatic retries.
* [Delivery operations](guides/delivery-operations.md) add audited recovery, state counts, replay cutoffs, and cleanup callbacks.
* [Java web push](guides/web-push.md) prepares encrypted requests with the optional SDK and records provider responses without treating acceptance as a read receipt.

Apply the [upgrade guide](upgrade-guide.md) before using the new storage features. The MySQL delivery timestamp migration keeps newly queued work from rounding into the next second.

These docs describe the candidate in [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2). Check [ForgeBox](https://forgebox.io/view/megaphone) for publication before updating a consumer.


## 1.0.4

* [**SlackProvider**](providers/slackprovider/): Use default [`SlackRoute`](providers/slackprovider/#routenotificationforslack) object if none is returned.

## 1.0.3

* Only invoke routing methods ([`routeFor{Provider}`](reference/baseprovider/#routenotificationfor)) if they exist.

## 1.0.2

* Ensure optional dependencies are activated when registering channels.

## 1.0.1

#### [EmailProvider](providers/slackprovider/#configuration)

* Use the default mailer from `cbMailServices` if no default mailer is configured for Megaphone.
* Log mail `send` successes to `info` and errors to `error` by default.

## 1.0.0

### Initial Release!
