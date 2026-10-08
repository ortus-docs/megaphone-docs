# Module Tests

Run the tests from a checkout of the [Megaphone module](https://github.com/coldbox-modules/megaphone), rather than an installed application dependency. Install the development dependencies with `box install`.

The integration tests need a dedicated MySQL database. Set `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD` for that database using the module's `.env.example`. Provision it before running the migrations. Keep tests out of your application or production database.

```bash
box install commandbox-migrations
box migrate install
box migrate up
box task run taskFile=tasks/InstallWebPushSDK.cfc
box server start
box testbox run
```

Use Java 11+ for the optional push SDK tests. The SDK installer downloads and verifies the JAR; the push preparation tests use transport fixtures and don't send to real devices. Migrations run before the suite, and persistence specs roll back their test writes.

The 1.1.0 candidate passed the CI matrix on Lucee 5/6 and Adobe ColdFusion 2021/2023, each with ColdBox 6/7. The four existing ColdBox 6 skips remain. See [module PR #2](https://github.com/coldbox-modules/megaphone/pull/2) for the current checks.

Your app still needs tests for its authorization constraints, preference scopes, queue transactions, worker handoff, provider callbacks, and browser enrollment. Verify actual providers and devices separately; passing the module suite doesn't establish those integrations.
