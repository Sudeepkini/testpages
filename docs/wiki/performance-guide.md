# Why care about performance? 🚀

One of the company's missions is to provide an app that will be preferred among competitors.

Having a performant app is one of the keys elements of user satisfaction.

# Performance Tips

- Nested views
  - Make sure that you use a less nested view. Take advantage of `ConstraintLayout`. But also if you build reusable layouts, take into consideration that they can be used inside `ConstraintLayout`, and that can also create potential perf issues.

- Serialize/deserialize on the main thread
  - Avoid serializing/deserializing on the main thread.

- Map data models to presentation models on the main thread
  - Avoid mapping data models to presentation models on the main thread

- Inflate complex views
  - View inflation takes the most part of the view creation. \
Use [ViewStub](https://developer.android.com/reference/android/view/ViewStub) for views that are feature flagged, or views that maybe will not be shown at all like: no internet connection, no restaurants, etc.

- RxJava
  - `Observable.just(heavyWork()).subscribeOn(Schedulers.io())` \
Even if we do subscribe on the IO thread, the `heavyWork()` will be executed on the main thread. We have to use [defer operator](https://reactivex.io/documentation/operators/defer.html)

- Unnecessary initialization of modules
  - Initialize your module only when it's required

- Complex SVGs
  - Having complex vector drawables are heavy to render

- Initialize DI components on a background thread
  - `Lazy<Dependency>` - for heavy dependencies try to get the instance of it on the background thread.. \When we deal with lazy dependencies, ideally we will do `dependency.get()` on the background thread. This will generate a dependency graph for this dependency off the main thread

- Long blocking animations
  - Avoid long and blocking animations
- [Coroutines mistakes we make](https://www.lukaslechner.com/7-common-mistakes-you-might-be-making-when-using-kotlin-coroutines/)
- Resources should be properly disposed after usage to avoid resource leaks using try/catch/finally pattern or `Closable.use()` extension method from the Kotlin stdlib
- Use `com.deliveryhero.commons.fastLazy` instead of lazy if you know what thread you’ll be executing on, and don’t worry about synchronization
- Avoid refreshing the view if the data is not changed
  - For RecyclerViews implement proper `DiffUtil.ItemCallback.areItemsTheSame()` and
   `DiffUtil.ItemCallback.areContentsTheSame()` methods, also utilize the ability to send payload
   indicating what exactly has been changed by overriding the `DiffUtil.ItemCallback.getChangePayload()`
   method
  - Use RxJava's `distinctUntilChanged()` operator, `Transformations.distinctUntilChanged()` for LiveData,
   or StateFlows (where data is already conflated using the same technique) to avoid emitting
   duplicate items - *please don't forget to properly implement items' `equals()` method or use data classes*

# Performance Troubleshooting

Measuring performance is hard. A lot of factors can affect the stability of the results. Always make sure that you are measuring performance on a real device and not an emulator

## What affects the performance results?

- [Debuggable overhead](https://youtu.be/ZffMCJdA5Qc?t=634)
  - Make sure that the app is NOT debuggable. Debuggable results can [vary from 0 to 80%](https://youtu.be/ZffMCJdA5Qc?t=634). It’s impossible to make conclusions if we have a lot of false positives.

- [CPU Thermal throttling](https://youtu.be/ZffMCJdA5Qc?t=952)
  - Make sure that the device is not hot/warm. CPU has the capability to protect itself and slows down its frequency.

- Low battery
  - Make sure that the battery is not low, or device is in power-saving mode

## Tools to detect performance issues

- [Profile GPU Rendering](https://developer.android.com/topic/performance/rendering/inspect-gpu-rendering#profile_rendering)
  - This tool provides an easy way to see if the screen rendering performance is good or bad
- [Overdraw](https://developer.android.com/topic/performance/rendering/inspect-gpu-rendering#debug_overdraw)
  - A tool that visualizes overdraw, and we are able to identify if we do unnecessary work. [How to fix](https://developer.android.com/topic/performance/rendering/overdraw#fixing) overdraw
- [Profile the code](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/performance-guide/)
  - To understand what is happening in the code and to identify slow methods, the best thing is to method trace.\
You can read our wiki on how to record a trace. After you pull the trace from the device you can open it with the Android Studio profiler.\
[Documentation](https://developer.android.com/studio/profile/cpu-profiler#overview) how to use the profiler\
[Inspect traces](https://developer.android.com/studio/profile/inspect-traces) with android studio\
[Great article](https://dev.to/pyricau/android-vitals-profiling-app-startup-32ek) by P.Y.
- [Firebase Performance - Screen Rendering](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/trends)
  - Monitor slow and frozen frames for activities and fragments. We are building a dashboard about this
- [Memory Profiling](https://developer.android.com/studio/profile/memory-profiler#overview)
  - A memory monitor enables you to see the memory application consumption at a given moment. You can dump the memory and inspect it ([only when app debuggable](https://developer.android.com/studio/profile#profileable-apps))
- [System trace](https://developer.android.com/topic/performance/tracing)
  - You can record device system activity into a trace file. It’s useful to spot performance issues like UI junk. Read [here more](https://developer.android.com/topic/performance/tracing/on-device) on how to record them on device

## Performance on CI

We are working on tooling to help us identify performance issues. And these tools are evolving all the time

- [Performance checks on CI](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/ci-performance-checks/)
We are observing method execution in performance tests. The idea is to identify all the bottlenecks in the code execution. Here is the [dashboard](https://datastudio.google.com/u/0/reporting/c6fc3b7b-8d37-4b4e-b913-fd57eb410d8f/page/p_u8xingfypc).

- [Memory leaks on CI](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/ci-performance-checks/)
  - We are observing memory leaks that are appearing in the app using [Leakcanary](https://square.github.io/leakcanary/). Soon we're gonna have a dashboard to have a better visualization of them.\ [Read more](https://docs.google.com/document/d/1PbsUlrdR7GwVUZF0Oe1Tv8ti8kPqhLnKhWXnldZt5Mw/edit?usp=sharing) on how we detect and report memory leaks

# Memory leaks

We detect memory leaks in CI using Leakcanary. We report them and save them in BigQuery.

You can read more about how we do it in the `Memory leaks on CI` section.

### Manual leak detection

Currently, you have to manually enable it. Find the usage `LeakCanaryUtils.init()` and pass true.

You can also try to reproduce the leak, [dump the heap](https://developer.android.com/studio/profile/memory-profiler#overview), and check for memory leaks.

# APK Size

App size is an important metric that directly correlates with [business metrics ](https://medium.com/googleplaydev/shrinking-apks-growing-installs-5d3fcba23ce2)like install conversion rate.

### Tool to detect app size regression

Currently, on Android, we have a workflow that checks for app size regression app size regressions a few times per day.

If a regression is detected will post a message in `#pd-tech-android`

![Android app size regression slack notification](https://i.ibb.co/RQydXWZ/Screenshot-2022-03-29-at-11-15-17.png "Android app size regression slack notification")


With every detected regression we generate a report, and you can see the report in the slack notification. Download report button ([example](https://storage.cloud.google.com/android-ta/104625/report.json)).

The report contains useful information to understand what was the reason for the regression, and which commit introduced it.

Report content:

```kotlin
involved_commits   //commits involved in the regression
commits_diff_file  //git diff with changes in all build.gradle / libs.versions.toml
diffuse_apk_file   //diffuse report that shows dex, arcs, res etc. difference
build_info.        //info about the build
apk_diff_size_kb   //diff between current and prev app size in kb
current_apk_path   //current apk download url
previous_apk_path  //previous apk download url
```
### Upcoming tools

We are working on detecting app size regression with every PR. Here you can [read more.](https://docs.google.com/document/d/1PZ_hh73r2TOJMuaHg4WYk9Yj2UxnrN2FvVVJXTZk0Jg/edit#)

### Best practices

* Be aware of the third-party libraries that you plan to use. Take in consideration the library size, and try to find a more lightweight one, or ask yourself if we really need it.
* Use webP image format. It is the [preferred format](https://developer.android.com/topic/performance/network-xfer#webp). Always challenge designers to provide optimized images.
* Use Lottie instead of gifs. Lottie has lots of advantages, some of them are 600% smaller than gif, 10x faster, resolution-independent, scalable, etc. Here you can [read more](https://lottiefiles.com/lottie)

## Measuring performance in production apps

We are using[ Firebase Performance](https://firebase.google.com/docs/perf-mon) to measure our KPIs.

We have a wrapper over Firebase performance, that makes it abstract and enables us to hide the implementation.
Repository for our [performance-analytics](https://github.com/deliveryhero/dh-mob-performance-analytics-android)

## Performance KPIs

### App start metrics

We collect a bunch of metrics that tell us how the app startup is performing.

* [app_cold_start](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_cold_start) (app starting from scratch, app is not in stack. [Read more](https://developer.android.com/topic/performance/vitals/launch-time#cold))
* [app_warm_start](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_warm_start) (app was already started, opened from stack for example. [Read more](https://developer.android.com/topic/performance/vitals/launch-time#warm))
* [app_start_to_interactive](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_start_to_interactive) (cold app startup until home screen is interactive)
* [app_start_to_launcher](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_start_to_launcher) (cold app startup until we see launcher screen)
* [app_di_init](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_di_init) (initialization of main DI component)
* [app_start_to_first_screen](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/app_start_to_first_screen) (cold app startup until we see launcher screen)

### Screen to interactive metrics

Every activity/fragment needs to track [time to interactive](#time-to-interactive). Tracking this information is a manual process. You can read the [documentation](https://github.com/deliveryhero/dh-mob-performance-analytics-android#screen-tracker) on how to track it.

Example of [homeScreenToInteractive](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/DURATION_TRACE/homescreenStartToInteractive)

You can see the pandora [dashboard for screen start to interactive](https://datastudio.google.com/u/0/reporting/114ac93b-9454-4d3d-b198-74d1c08dd1ca/page/I1pAC)


### Frozen frames

This metric is the percentage of frames that were frozen for a specific screen. Specifically, this metric is the percentage of screen instances during which more than 0.1% of frames took longer than 700 ms to render. [Llink](https://firebase.google.com/docs/perf-mon/screen-traces?platform=ios#frozen-frames)

The dashboard is in progress. Example of a [trace](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/SCREEN_TRACE/_st_AddressDropdownFragment)


### Slow frames

This metric is the percentage of frames that were slow to render for a specific screen. Specifically, this metric is the percentage of screen instances during which more than 50% of frames took longer than 16 ms to render. [Link](https://firebase.google.com/docs/perf-mon/screen-traces?platform=ios#slow-rendering-frames)

The dashboard is in progress. Example of a [trace](https://console.firebase.google.com/u/0/project/android-foodora/performance/app/android:com.global.foodpanda.android/metrics/trace/SCREEN_TRACE/_st_AddressDropdownFragment)

# How to profile the code

In order to understand what is happening in the code and to identify slow methods, the best thing is to profile the code and record a method trace.

### Before you start profiling

It is important to set up the environment for profiling and getting the most stable results.

1. Disable debuggability. In `app/build.gradle.kts` set `isProfileableBuild = true`. Or you can use **_brandBenchmark_** variant
2. Kill all apps in the stack
3. Go through the flow that you want to test, so we make sure that the code will be pre-compiled and not JIT-ed.
4. Make sure that the device is not warm/hot
5. With every run, force close the app to ensure stable results and skip cache.
6. Don't use an emulator. Use real device

### Record trace with Android Studio (Bumblebee)

Recording a trace on a non-debuggable app with Android Studio is only available on Android Studio Bumblebee and above.

1. Make sure you disable debuggability
2. Open profiler View > Tool Windows > Profiler
3. Start a new session using the (+) button
4. Choose your device > Other profitable processes > app package
5. Click on CPU timeline
6. Choose Callstack Sample Recording and press record

### Record trace with MethodTracing

MethodTracking API is using Android Debug API. It allows us to record traces programmatically only for part of the code you are interested in. We are using method sampling under the hood.

>You can not record multiple nested traces. Only one trace at a time.

```kotlin
performance-test/src/main/java/com/deliveryhero/performance/test/trace/MethodTracer.kt

 fun startSampling(traceName: String) //generates .trace(file) in internal storage of the device
 fun stopSampling()
```

Generates trace file in the device under the path: `/storage/emulated/0/Download/dh-app-trace/$traceName-yyyy-MM-dd_HH-mm-ss_SSS.trace`

To pull the traces from the device use:

```kotlin
   scripts/pull-trace-from-device.sh
```

The script will pull trace files from the device, and copy them under `/pd-mob-b2c-android/scripts/test-traces/dh-app-trace`.

### How to analyze a trace

Once you have the trace you can open it with Android Studio Profiler and analyze it.

1. Open Android Studio Profiler view. View > Tool Windows > Profiler
2. Sessions (+) > Load from file

![Android Studio Profiler Trace Open](https://i.ibb.co/CnyCCsd/Screenshot-2022-03-29-at-11-22-03.png "Android Studio Profiler Trace Open")

You have an overview of all threads and their traces.

The thread that you will most likely be interested in is the  `main` thread.

Select the `main` and then you will see the Call chart. You can navigate the trace with your keyboard:

>* W - zoom in
>* S - zoom out
>* A - move left
>* D - move right

Here are some useful links with explanations:

* [Documentation](https://developer.android.com/studio/profile/inspect-traces) of the Android Studio Profiler components (Call Chart, Flame Chart)
* [Documentation](https://developer.android.com/studio/profile/cpu-profiler#overview) for basic profiler functionality
* [Video](https://www.youtube.com/watch?v=v4kCRZ_O4Lc) explanation that will help you understand Profiler components (Call Chart, Flame Chart…)

### Default Screen Trace and its metrics

The `ScreenPerfMetricContainer` class create a trace for each activity or fragment that get created. All the related metrics for specific page will be send under one trace for screen in firebase. Trace name does have prefix of `sm_` and then screen name. You can find it under custom traces in firebase performance
Also, for using the sdk you can find documents in the codebase for each method but because there's limit for number of metric per trace(32) do not add any metric without arrangement with performance squad

## Screen to navigation respond

screen trace right now include this metric for every page. it keep track of the time and add the total time as metric to related page screen trace with `sntr_ms` key
If you need more information about what is that and how we implement it check this [[article]](https://dev.to/pyricau/android-vitals-tap-response-time-19mj) from py
You can manually add the source screen (the screen that user been there before and clicked on button to get here) as attribute to screen trace with below line:

`SourceTracker.sourceScreen = "sourceScreen"`
Consider you should add this line when looper is handling onClickListener and do not post it to handler. Otherwise it may get deleted later.

## Performance resources

* Series of articles from PY. Highly recommended to read them [[LINK]](https://dev.to/pyricau/android-vitals-profiling-app-startup-32ek)
* Video explanation on how Android Renders the UI [[LINK]](https://www.youtube.com/watch?v=zdQRIYOST64)
* Video explaining profilers and how to use them [[LINK]](https://www.youtube.com/watch?v=v4kCRZ_O4Lc)

## Appendix

<h3 id="time-to-interactive">Time to interactive</h3>

Time to interactive is the amount of time it takes for the screen to become fully interactive
