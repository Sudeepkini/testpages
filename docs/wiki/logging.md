# Logging

## Timber

We use [Timber](https://github.com/JakeWharton/timber) as our only logging framework across all modules in the codebase.
It's flexible and configurable, and the whole logging configuration could be easily overridden for every build type / flavor etc.
It allows us to log breadcrumbs and handled exceptions to Sentry or any other error monitoring system seamlessly, 
as well as to stream same logs to logcat for debugging purposes on debug builds.

### How do I use it in my module?
Simply add a dependency on Timber and you’re good to go:
```
implementation "com.jakewharton.timber:timber:${rootProject.timberVersion}"
```

We do not need to do any configuration or DI injection to use it. Just call it from the code:
```kotlin
Timber.e(throwable, breadcrumb) // or any other variation as the case may be
```

## Log levels

Always use appropriate log levels to avoid leaking sensitive information and prevent abusing Sentry quota.

### Verbose

Useful for tracking bugs while developing, but should not be ever committed into VCS.

### Debug

Should be used for troubleshooting, e.g. dumping whole data objects or network calls.

### Info

Should be used for logging all notable user-driven or system-generated events that are not considered as errors / warnings.
Could also be accompanied by Debug-level log message with more debugging information about the event (e.g. whole object dump).
Should be the most used and informative log level in the application.

Logs with `INFO` level will be sent to Sentry as breadcrumbs.

### Warning

Useful for logging events that could potentially become an error, but still are recoverable, e.g. API call timeouts.

Logs with `WARN` level _without throwable_ will be sent to Sentry as breadcrumbs.

Logs with `WARN` level _and throwable_ will be sent to Sentry as non-fatal issues with `level:warning`.

### Error

Should be used for logging unrecoverable exceptions or fatal states.
In most cases the exception itself should also be passed to the Timber call, 
so it could be logged into Sentry as the cause.

Logs with `ERROR` level _without throwable_ will be sent to Sentry as breadcrumbs.

Logs with `ERROR` level _and throwable_ will be sent to Sentry as non-fatal issues with `level:error`.

### WTF

Should not be used inside the production code.

### Log levels summary

- Any log below `INFO` will not be sent to Crashlytics or Sentry, 
e.g `Timber.v()` and `Timber.d()` will send logs to logcat only as they are intended for debug purposes.
- Logs without throwable (even with `WARN` or `ERROR` levels) will be sent to Sentry only as breadcrumbs,
not as issues, e.g. `Timber.w("Message")` will be added as a Sentry breadcrumb, 
but `Timber.w(SomeException(), "Message")` will be reported as an issue.
- Logging breadcrumb and throwable is done in a single call, no need to do `logBreadCrumb()` + `logException()` anymore.
- `WARNING`+ level should be used only for logging erroneous (from the development point of view) situations,
not for product-related events, because those events will be sent to Sentry and spend the quota.

## Meaningful messages

Log messages should be meaningful, and the context should be also present.
It's easy to understand the meaning of the log message while you're developing something,
but the context is generally unknown for other developers, so you should always include context inside your log message.

## Generic exceptions

Avoid logging generic `Throwable` or `Exception` instances, it's more difficult to search, ignore or investigate 
such exceptions on Sentry. Always prefer to create an instance of the exception class specifically created for the use case
(e.g. prefer `FirebaseTokenRegistrationFailed()` over the generic `IOException("Failed to register Firebase token")`).

## Independent messages

Log messages should not depend on previous messages' content.
In asynchronous application with different log levels messages are not always appear one after another,
so you should always provide the context for every log message, assuming the previous messages' content is not visible.

## Tagging

Timber takes care of the proper tagging using (not fully qualified) class name as a tag,
but also provides the ability to override tag for a single log message (e.g. `Timber.tag("SomeFlow").e(exception)`),
it's useful for introducing custom tags (e.g. for separate flows in the app).

Timber -> Sentry integration also partially supports custom Timber tags, this custom Timber tag will be 
added as a `TimberTag` custom tag for the created Sentry event.

## Parsable message format

Prefer parsable log messages in addition to proper tagging and prefixing,
values that could contain spaces and other special characters should be quoted and, if necessary, properly escaped.

Structured log messages (e.g. JSON-like `key=value` or YAML-like `key:value`) 
are generally more preferable than free-form text messages
(e.g. `action=car_park,car_make="Alfa Romeo",car_model="Alfa 90"` is better than `User parked Alfa Romeo Alfa 90 car`).

## Sensitive information

Even with logging disabled in release build type, sensitive information (e.g. passwords, tokens etc.) should not be logged
with log levels `INFO` and higher.

Ensure that only essential and properly sanitized breadcrumbs are sent to Sentry.

## Parametrized messages vs concatenation

Timber parametrized logging methods (e.g. `public void i(String message, Object... args)`) 
should be used instead of string concatenation.
Parametrized methods are faster (as the resulting log message isn't formatted if it's not loggable).
Timber also includes a set of lint rules for checking proper usage.

## Avoid side-effects

Carefully decide what are you going to log, avoid any possible side-effects that could be produced by logging. 
For example, logging could trigger some lazy initialization or state change for stateful classes
(see the OkHttp's `ResponseBody.string()` method which consumes and closes the stream, 
making it effectively unusable for other consumers).

## Avoid using Android `Log.*` methods or `java.util.logging.*` loggers

## **Never** do `exception.printStackTrace()`
