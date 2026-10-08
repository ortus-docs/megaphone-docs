# Optional Java Web Push transport

`WebPushTransport` uses Java 11+ and the optional
[zerodep-web-push-java SDK](https://github.com/st-user/zerodep-web-push-java).
Run this from the installed module directory:

```sh
box task run taskFile=tasks/InstallWebPushSDK.cfc
```

The task downloads version 2.1.5 from Maven Central and checks SHA-256
`1337acba24004f2a702275b676eb3862e402c8775d3031b26464f0b18d3ba989`
before saving the JAR under `resources/java/webpush`. If the file already exists,
it checks that file. Add this step to your deployment build if you use push;
the regular Megaphone install doesn't download the SDK.

Load that directory with your application's Java loader. On Adobe 2021 and
2023, add it to `this.javaSettings.loadPaths` and have your class factory call
`createObject( "java", className )`. Pass its `create`
class factory, the SDK's VAPID key pair, a contact subject (`mailto:` or HTTPS),
and the push-service DNS hosts you trust to `new WebPushTransport(...)`.
Host entries can be exact names or `*.example.com` patterns. The wildcard matches
subdomains, not the parent domain. Configure these hosts in your app; don't take
the list from browser input.

The default client disables redirects and sets connect and request timeouts.
You can pass an `httpClient` to use your own transport or a test fixture. It must
keep those timeout and redirect rules and must not retry automatically.

`prepare(subscription, payload, ttlSeconds=3600)` validates the HTTPS endpoint,
keys, and single-record UTF-8 payload size, then signs with VAPID and encrypts
with `aes128gcm`. It doesn't make a network request. TTL can be zero through seven
days. Prepare the request in your eligibility callback, before marking transport
as started, and recheck registration ownership and notification access there.
Keep the prepared request internal; its URI and headers contain private data.

After the dispatcher marks transport as started, `sendPrepared(request)` sends
once and discards the response body. It returns a `WebPushResponsePolicy` outcome,
or ambiguous acceptance if transport fails. When `retireSubscription=true`,
retire that recipient's registration. One failed or expired device shouldn't
cancel another device's delivery. Sending push doesn't create inbox entries or
change preferences.

After installing the SDK, you can run
`tests.specs.integration.WebPushPreparationSpec` to check SDK request preparation
and transport fixtures. It doesn't send to real devices. Test your actual push
providers, service worker, enrollment flow, and devices separately.
