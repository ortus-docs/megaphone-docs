# Module Tests

Want to run Megaphone's tests locally? Start with a checkout of the [module repository](https://github.com/coldbox-modules/megaphone), and install its development dependencies with `box install`. Use the Java version required by your selected engine. The checkout defaults to Adobe ColdFusion 2023 on JDK 17.

### Setting up the test database

The integration suite needs a dedicated MySQL database. Create it first, then use `.env.example` to configure `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD`.

The checkout defines migration managers for Megaphone and cbq. Run both before starting the suite:

```bash
box install commandbox-migrations
box migrate install
box migrate up
box migrate install manager=cbq
box migrate up manager=cbq
box server start
box testbox run
```

The cbq integration specs insert real database jobs and execute their serialized delivery jobs. Test workers stay disabled so the suite controls when a job runs. Persistence specs roll back their test writes. Digest concurrency specs commit isolated fixtures so separate connections exercise the actual row locks, then remove only their own records. Use a database set aside for this suite, separate from your application's data.

### Testing BoxLang

Install the CommandBox BoxLang integration and the framework versions used by these jobs:

```bash
box install commandbox-boxlang
box install coldbox@8 --noSave
box install testbox@6 --noSave
box install qb@14 --noSave
box install cbq@7 --noSave
box install hyper@8 --noSave
box server start serverConfigFile=server-boxlang.json --noSaveSettings
box testbox run
box server stop serverConfigFile=server-boxlang.json
```

Use `server-boxlang-cfml.json` for the CFML compatibility run. The configurations use separate engine homes so the compatibility module from one run does not affect the other. Both use the same test suite and database setup above.

### Testing Web Push

For the SDK tests, use Java 11+. The checkout's `box install` command installs the pinned JAR dependency through CommandBox's JAR endpoint; no separate SDK setup is needed. Push preparation tests load the real SDK and use fixture HTTP clients, so they don't send notifications to real devices.

### Testing your application

The module's engine matrix covers Lucee 5/6 and Adobe ColdFusion 2021/2023 with ColdBox 6/7. It uses JDK 11 for Lucee 5/6 and Adobe 2021, and JDK 17 for Adobe 2023. BoxLang jobs run both native and CFML compatibility modes on JDK 21 with ColdBox 8, TestBox 6, qb 14, cbq 7, and Hyper 8. The compatibility server installs `bx-compat-cfml`; the native server leaves it out. Both install the MySQL, mail, and ESAPI runtime modules. Scheduled jobs also cover both modes on bleeding-edge BoxLang and ColdBox. A runtime spec checks that the selected mode is actually loaded. See [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2) for the current check results.

Your application's tests should also exercise its authorization constraints, preference scopes, queue transactions, worker handoff, and Provider callbacks. For Web Push, check browser enrollment and delivery on your actual devices as part of verifying the integration.

The development checkout installs cbq for the queue integration suite. To exercise the optional-dependency checks, move `modules/cbq` out of the module discovery path, run the `tests.specs.integration.OptionalCBQSpec` bundle, then restore it. That bundle verifies startup, synchronous notifications, custom adapters, polling, and the `Megaphone.Queue.MissingCBQ` exception with cbq absent.

### Testing email digests

The suite covers module registration and storage configuration, collection through Notifications, independent named subscriptions, cutoff membership, native ColdBox schedules and task templates, consent and eligibility changes, mode changes, and simultaneous workers. Fixture email Providers return simulated receipts without sending live mail.

`tasks/VerifyPostgres` includes the digest and storage suites for an explicitly provisioned disposable PostgreSQL database. When other module repositories live beside the checkout, set the test-only JVM property `megaphone.testModuleRoot` to an isolated parent containing only this checkout as `megaphone`; the HTTP test application uses that directory for external module discovery.
