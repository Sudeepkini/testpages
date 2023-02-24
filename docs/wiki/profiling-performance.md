# Intro
Profiling is very tricky, there are a lot of factors that affect the results. And coming up with stable results it's hard.
The best way to verify the results is to run a benchmark multiple times.

# Before you start measuring

1. Disable debuggability. In `app/build.gradle.kts` set `val isProfileableBuild = true`
2. Kill all apps in the stack
3. Go through the flow that you want to test, so we make sure that the code will be pre-compiled and not JIT-ed.
4. Make sure that the device is not warm/hot
5. With every run, force close the app to ensure stable results.
6. Don't use an emulator. Use real device

# Method tracing using Android Studio (bumblebee)

1. Make sure you disable debuggability
2. Use Android Studio (bumblebee)
3. Open profiler
4. Start new session using (+) button
5. Choose your device > Other profitable processes > select the app
6. Click on CPU timeline
7. Choose `Callstack Sample Recording` and press record

# Method tracing using Debug API

If you want to trace some part of the code you can use `performance-test/src/main/java/com/deliveryhero/performance/test/trace/MethodTracer.kt`


Call `MethodTracer.startSampling(traceName: String)` before the code that you want to profile (generates trace in internal storage)


Call `MethodTracer.stopSampling()` when you are done with sampling.


This will generate a `$traceName-yyyy-MM-dd_HH-mm-ss_SSS.trace` file in the device. Path: `/storage/emulated/0/Download/dh-app-trace/$traceName-yyyy-MM-dd_HH-mm-ss_SSS.trace`
To pull all the traces from the device use: `scripts/pull-trace-from-device.sh`. The script will pull the files in `/pd-mob-b2c-android/scripts/test-traces/dh-app-trace` and will remove the traces from the device.


If you have questions ping `@pd-squad-app-performance-reliability` squad
