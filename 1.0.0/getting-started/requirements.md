# Requirements

Megaphone requires JDK 11 or newer, ColdBox 6+, and either Adobe ColdFusion 2021+ or Lucee 5+.

Your engine may need a newer Java version. Adobe ColdFusion 2023 uses [Java 17](https://helpx.adobe.com/pdf/coldfusion2023-support-matrix.pdf), and BoxLang requires [Java 21](https://boxlang.ortusbooks.com/getting-started/installation). Use the higher requirement for your application.

CommandBox installs the [Web Push Provider's](../providers/webpushprovider.md) declared SDK JAR dependency with the module. The SDK is loaded only when you use Web Push; other channels don't need push credentials or configuration.
