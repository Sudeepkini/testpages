# Network replay

## What is network replay?

Network replay is a way to automatically store all outgoing network requests and real network responses
in device's storage and replay them instead of making real network requests for subsequent sessions.

Currently network replay is used for macrobenchmark tests to get more stable and predictable results
by eliminating potential network delay. Other potential applications include UI tests, debugging etc.

## How does it work?

### Gradle plugin

Network replay feature is implemented using a separate Gradle plugin
(`com.deliveryhero.plugin.network-instrumentation`) located in the `:gradle-project-plugins` and applied to the
`:app` module. Once enabled, this plugin uses the [bytecode manipulation](#bytecode-manipulation) to inject
the [specific code](#network-replay-module) into all network clients.

### Bytecode manipulation

This feature uses the [bytecode manipulation](https://www.infoq.com/articles/Living-Matrix-Bytecode-Manipulation/)
framework called [ASM](https://asm.ow2.io/) to automatically attach custom OkHttp interceptor to all `OkHttpClient`
instances in the project. It's attached not only to our `OkHttpClient` instance provided by the
`PandoraHttpClientBuilder#build`, but also to all 3rd-party client instances used by other dependencies,
including FwF, Sentry, Qualtrics etc.

### Network replay module

Once the `instrumented` property is passed, Gradle plugin will automatically include the `:network-instrumentation`
module as an implementation dependency for the `:app`. This module contains the actual replaying interceptor
(`/network-instrumentation/src/main/java/com/deliveryhero/network/instrumentation/DumpingInterceptor.kt`) and
dumping/replaying logic.

## How to enable network replay?

- use proper app build variant - currently network replay works *only* with `foodpandaDebug` and
  `foodpandaBenchmark` build variants due to the hardcoded storage path;
- pass the `instrumented` property to the Gradle build, e.g. by invoking following command
  `./gradlew assembleFoodpandaDebug -Pinstrumented`.

## Using network replay

Once the build with enabled network replay is installed on the device and started, it'll automatically create the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/` directory
for all network replay-related files.

First run of the app will create the session directory
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/`
with journal file, request body files, request header files, response body files and response header files.

All subsequent runs will try to use stored network responses instead of the real ones if possible, using
**the request network method** and **the request URL** to match the request to the stored response.
If the match isn't found (or the request is blacklisted), replay framework will fall back to using the real
network responses instead. If many responses match the requested method and URL, **only the first one will
be replayed** for all requests.

### Logs

All logged operations are stored in logfiles
(`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/log_${start}.log`),
where `${start}` is epoch seconds value of the beginning of each session,
e.g. `/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/log_1676390390.log`
.

Log file is actually just a comma-delimited CSV file containing a number of columns:

- date/time of the entry in the device's default timezone;
- log level (one of the following: `VERBOSE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `WTF`) of the entry;
- text message of the entry, quoted with the `"` character;
- optional associated throwable converted to string (something like `java.lang.NullPointerException: message`)
  quoted with the `"` character.

For example:

```
2023-02-14T17:00:09.350+01:00[Europe/Berlin],INFO,"Dumping Request{method=POST, url=https://px-conf.perimeterx.net/api/v1/mobile}",
```

or

```
2023-02-14T17:02:00.123+01:00[Europe/Berlin],ERROR,"Duplicate requests for DefaultKey(method=GET, url=https://static.fd-api.com/feature-config/staging/us/config.json?v=266501)",
```

Logger will also duplicate the log message to the LogCat using the same priority and the `NetworkInstrumentation` tag,
e.g.

```
2023-02-14 17:02:17.018 7715-7853/com.global.foodpanda.android.dev I/NetworkInstrumentation: Replaying Request{method=POST, url=https://perseus-stg.deliveryhero.net/v1/insert/pandora/hits, tags={class retrofit2.Invocation=com.deliveryhero.perseus.PerseusHitsApi.sendEventsHits() [pandora, com.deliveryhero.perseus.HitsRequest@9d7f7d2]}}
```

### Journals

Journal file is the CSV file representing the session with the information about all requests in the session.
One session has one journal file, e.g. by default the only session stored before replaying will have the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/journal.csv`
journal file.
Each journal entry is on a separate line in the journal file and contains a number of columns:

- identifier of the request (auto-incrementing integer unique inside one session);
- moment of the request start - before performing any network actions;
- moment of the request end - after the response with headers was received, but the response body is
  not necessarily yet fully received;
- HTTP method of the request;
- URL of the request;
- HTTP status code of the response;
- response protocol (HTTP/1, HTTP/2 etc.);
- HTTP status message of the response;
- request body content type if any;
- response body content type if any;
- if the request body exists;
- if the response body exists.

For example:

```
8,1676390394599,1676390394708,GET,https://static.fd-api.com/feature-config/staging/us/config.json?v=266501,403,h2,,,application/xml,false,true
```

### Request headers

Request headers are stored in CSV files in the session directory
(`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_${sessionId}/request_headers_${requestId}.csv`)
, e.g. headers of the request with id 1 of the default first session are stored in the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/request_headers_1.csv`
file.

Each line of the request headers file is a comma-separated key-value pair of the request header, e.g.
`User-Agent,Android-app-23.4.0(266501)`.

### Request bodies

Request bodies are stored in binary files in the session directory
(`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_${sessionId}/request_body_${requestId}.bin`)
, e.g. the body of the request with id 1 of the default first session are stored in the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/request_body_1.bin`
file.

Request body file is present only if the original HTTP request body is present in the request and it's not one-shot.

### Response headers

Response headers are stored in CSV files in the session directory
(`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_${sessionId}/response_headers_${requestId}.csv`)
, e.g. headers of the response corresponding to the request with id 1 of the default first session are stored in the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/response_headers_1.csv`
file.

Each line of the response headers file is a comma-separated key-value pair of the response header, e.g.
`cache-control,"no-cache, private"`.

### Response bodies

Response bodies are stored in binary files in the session directory
(`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_${sessionId}/response_body_${requestId}.bin`)
, e.g. the body of the response corresponding to the request with id 1 of the default first session are stored in the
`/storage/emulated/0/Android/data/com.global.foodpanda.android.dev/files/network_instrumentation/session_0/response_body_1.bin`
file.

### Blacklisting requests

Some requests can't be properly replayed using saved responses, because they use timestamps
(or timestamp hashed values) as part of the request / response pair, so saved responses can't be properly parsed
or interpreted, which can cause the app to malfunction. To avoid it, there's a mechanism for preventing some requests
from being replayed, always falling back to network. Currently those blacklisting rules can use request URL host
or the whole request URL, see the `com.deliveryhero.network.instrumentation.ReplayerKt#blacklistedHosts`
and `com.deliveryhero.network.instrumentation.ReplayerKt#blacklistWildcards`.

Currently blacklisting rules enforce following requests to fall back to network:

- Shakebugs;
- Perseus;
- Qualtrics;
- PerimeterX.
