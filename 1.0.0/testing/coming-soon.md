# Module Tests

Want to run Megaphone's tests locally? Use JDK 11, start with a checkout of the [module repository](https://github.com/coldbox-modules/megaphone), and install its development dependencies with `box install`.

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

The cbq integration specs insert real database jobs and execute their serialized delivery jobs. Test workers stay disabled so the suite controls when a job runs. Persistence specs roll back their test writes. Use a database set aside for this suite, separate from your application's data.

### Testing Web Push

For the SDK tests, use Java 11+. The checkout's `box install` command installs the pinned JAR dependency through CommandBox's JAR endpoint; no separate SDK setup is needed. Push preparation tests load the real SDK and use fixture HTTP clients, so they don't send notifications to real devices.

### Testing your application

The module's engine matrix covers Lucee 5/6 and Adobe ColdFusion 2021/2023 with ColdBox 6/7. See [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2) for the current check results.

Your application's tests should also exercise its authorization constraints, preference scopes, queue transactions, worker handoff, and Provider callbacks. For Web Push, check browser enrollment and delivery on your actual devices as part of verifying the integration.

The development checkout installs cbq for the queue integration suite. To exercise the optional-dependency checks, move `modules/cbq` out of the module discovery path, run the `tests.specs.integration.OptionalCBQSpec` bundle, then restore it. That bundle verifies startup, synchronous notifications, custom adapters, polling, and the `Megaphone.Queue.MissingCBQ` exception with cbq absent.
