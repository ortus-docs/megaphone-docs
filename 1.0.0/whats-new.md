# What's New?

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
