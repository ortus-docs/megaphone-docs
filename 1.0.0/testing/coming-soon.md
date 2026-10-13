# Module Tests

Want to run Megaphone's tests locally? Start with a checkout of the [module repository](https://github.com/coldbox-modules/megaphone) and install its development dependencies with `box install`.

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

For the optional SDK tests, use Java 11+ and run the SDK installer before the suite:

```bash
box task run taskFile=tasks/InstallWebPushSDK.cfc
```

The installer downloads the pinned JAR and verifies its checksum. Push preparation tests use transport fixtures, so they don't send notifications to real devices.

### Testing your application

The module's engine matrix covers Lucee 5/6 and Adobe ColdFusion 2021/2023 with ColdBox 6/7. See [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2) for the current check results.

Your application's tests should also exercise its authorization constraints, preference scopes, queue transactions, worker handoff, and Provider callbacks. For Web Push, check browser enrollment and delivery on your actual devices as part of verifying the integration.
