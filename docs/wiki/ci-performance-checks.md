# Introduction

Automated tools to collect and report app performance data. The idea is to take advantage of existing UI tests and enable performance checks. When the test is finished we collect the performance data and upload it to bigquery.

# How to enable it

- `PerformanceTest` annotation - is enabling performance checks on CI.
- `PerformanceType` enum - specify the type of the performance check (TRACE, STRICT_MODE, MEMORY_LEAK)

# Types of performance checks

### Trace
When the selected test is marked as `TRACE` test we do method tracing. It will record all methods(all code) that is executed while the test is running.
With this, we are able to see under the hood. It gives us detailed information and we can easily identify what is slow and focus on fixing that parts.
All this information is stored in bigquery and we will have a dashboard(TBD) separated by squad pointing where the biggest issues are.

- Used API: [Debug](https://developer.android.com/reference/android/os/Debug)
- Package: `com.deliveryhero.performance.test.trace`

### Strict mode
When the selected test is marked as `STRICT_MODE` test we start to record violations. That means we are recording things in the code that are considered a violation. Such things are: read from disk, write on disk, etc.
With this, we will be able to detect potential performance issues and focus on fixing them.
All this information is stored in bigquery and will be presented in a dashboard(TBD).

- Used API: [StrictMode](https://developer.android.com/reference/android/os/StrictMode)
- Package: `com.deliveryhero.performance.test.strictmode`

### Memory leak
When the selected test is marked as `MEMORY_LEAK` test we start to check for memory leaks. It will detect code that is causing memory issues. It will generate a report for that code and we can easily see what is the issue.
All this information is stored in bigquery and will be presented in a dashboard.

- Used API: [Leakcanary](https://square.github.io/leakcanary/)
- Package: `com.deliveryhero.performance.test.leak`

NOTE: `Performance-Check-Memory-Leak` workflow runs `RegressionTest` and all tests annotated with this annotation will be run.

# How often we run performance checks

We run performance checks nightly (once per day)

# Workflows

Only in the following workflows, we run performance checks:

- `Performance-Check-Memory-Leak` - Checks memory leaks running `RegressionTest` and uploads data to bigquery

- `Performance-check` - Runs all tests marked as `PerformanceTest` uploads perf data to bigquery

Both workflows post notification in `#pd-android-performance-notifications` if they finished with success or error

# How to test it locally

Make your app not `debuggable`. In `build.gradle` make this `val isProfileableBuild = true //hasProperty("profileable")`

Annotate your test with `PeformanceTest` and add what types of performance checks you want to be performed. After the test is finished reports will be generated in your device under `sdcard/performance`.
