 # Unit tests

* Unit tests are required with every new logic introduced in the codebase. Prioritize to unit test parts that contain business, presentation or data logic.
* Execution of Unit test should be lightning fast.
* For RxJava usage to override different schedulers in tests there are 2 rule solutions: `TestSchedulerRule` and `TrampolineSchedulerRule`. TBD the best approach and which one to use


# UI Instrumentation tests

Currently in the application we have Critical and Regression tests located in the main app that integrates all modules. Framework is setup and using Espresso together with IdllingResources.
IdllingResources are integrated with each `OkHttpClient` and `Rx2Idler` [library](https://github.com/square/RxIdler) handles IdllingResources for `RxJava` computation, io and new thread Schedulers thread pools.

**Pro tips**:

Whenever `debounce` is used in the code without specifying `Scheduler` its mandatory to add either `AndroidSchedulers.mainThread()` or `TimeScheduler.time()` (from commons module). With this there will be no flakiness in the tests execution because `debounce` without specific scheduler send it executes on `computation` thread.

ℹ️ ℹ️ ℹ️ **Debugging API issues in Allure for TA failures.** [video](https://drive.google.com/file/d/166BwlXta0o7PDAAPEILvTOi1e8UHOHoN/view?usp=sharing)

## In-Squad TA Development for New Features (Requirements & Responsibilities)

https://confluence.deliveryhero.com/pages/viewpage.action?spaceKey=GLOBAL&title=In-Squad+TA+Development+for+New+Features

### Running UI tests

To run all smoke tests
```
./gradlew connectedFoodpandaDebugAndroidTest -PESPRESSO_COUNTRIES=HK -PTEST_TYPES=SmokeTest
```
To run specific class or method tests
```
./gradlew connectedFoodpandaDebugAndroidTest -PESPRESSO_COUNTRIES=HK -Pandroid.testInstrumentationRunnerArguments.class=<full class path>#<method name>
```
or alternatively switch your IDE's flavor to `FoodPanda` and run AndroidTests

**TA Infra Team workshop** for using/creating tests on our platform:
https://github.com/deliveryhero/pd-android-ui-testing-workshop

* Every new feature introduced in the app should be covered with UI test.
* Test scenarios should be aligned with QA team.
* Avoid using `Thread.sleep()` in the instrumentation tests.


### Tracking event validator

When writing UI tests for your feature it is also recommended to take **Analytics** events into consideration and have
them as part of the assertions. `PandoraTestBase` has a property called `eventSentValidator` which you can use to assert
events are being sent. Sample usage is as follows:
```
@Test
fun sampleTest() {
    somePageObject.clickButton()
    eventSentValidator.assertEventTracked(
        EVENT_NAME,
        //any event attributes to verify
        mapOf(
            PARAM_NAME_A to "value_x",
            PARAM_NAME_B to "value_y"
        )
    )
}
```
One last note, if an event is being sent from the app but is **NOT** tracked by any `trackers` it will fail all `E2ETests`
but it won't affect `SmokeTests` so PRs won't be blocked.

# [Robo Test](https://firebase.google.com/docs/test-lab/android/robo-ux-test) (Firebase Test Lab)

> Robo test is a testing tool that is integrated with Firebase Test Lab. Robo test analyzes the structure of your app's UI and then explores it methodically, automatically simulating user activities. Unlike the UI/Application Exerciser Monkey test, Robo test always simulates the same user activities in the same order when you use it to test an app on a specific device configuration with the same settings. This lets you use Robo test to validate bug fixes and test for regressions in a way that isn't possible when testing with the UI/Application Exerciser Monkey test.

- We have a scheduled nightly Robo-test build on Bitrise that checks random flows in our app after following the steps mentioned in the [`robo-script.js`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/scripts/robo-script.js) file present in the `scripts` package.
- You can run a Robo-test of your own by selecting the branch and **Robo-Test** workflow on Bitrise.

### Configure feature-flags for Robo test

- Feature flags for LaunchDarkly or Firebase Remote Config can be overridden for a robo-test build in the [`RoboConfigsInterceptor.kt (debug)`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/configs/src/debug/java/com/deliveryhero/configs/RoboConfigsInterceptor.kt) file. This is especially useful when the flag isn't activated yet but the feature is ready for testing.

# Unit test coverage


### How do I check unit test coverage of my module?


Add list of files that need to ignored based on your package/project folder structure of module:

```
   junitJacoco {
       excludes = [
            '**/R.class',
            '**/R$*.class',
            '**/BuildConfig.*',
            '**/Manifest*.*',
            '**/*Test*.*',
            'android/**/*.*',
            '**/util/*',
            '**/tracking/*',
            '**/*Factory*.*',
            '**/**/*Factory*.*',
            '**/**/*Fragment*',
            '**/**/*ViewHolder*',
            '**/**/*Adapter*',
            '**/**/*Creator*',
            '**/**/*Activity*',
            '**/*Provider*',
            '**/*FeatureConfig*',
            '**/*$ViewInjector*.*',
            '**/di/*',
            '**/fastadapter/*'
    ]
}
```
**Please only exclude the files which cannot be tested and not which can be tested just for the sake of increasing percentage**

* Sync Gradle
* Open Gradle Window
* Go to tasks -> Reporting
* Find the jacoco task named **jacocoTestReportDebug** for the module you want to run and double click the task to run or you can also run in terminal `./gradlew jacocoTestReportDebug` in terminal


Check coverage report into the below folder:
    Your `project->build->reports->jacoco->debug->index.html`

# Memory Leaks

### Auto detect memory leaks on CI. How does it work?

We have enabled LeakCanary on TA and every `@CriticalTest` that is annotated with `@CheckMemoryLeaks` will be checked for memory leaks.

### Check memory leaks on RoboTest

With every robotest we check for memory leaks. If memory leaks were found during execution the test will fail(*) and will throw a `LeakDetectedException`.

(*) NOTE: Not every time the Robo test will fail if leaks were detected. Sometimes the test will report the test result before the heap analyses are done. In this case, the test will pass and we need to manually check if the test log contains leaks.

### Enable LeakCanary locally?

If you want to check for memory leaks locally you can do the following.

1. Add `@CheckMemoryLeaks` on the test that you want to check
2. Enable LeakCanary
`app/debug/de.foodora.android.FoodoraDebugUtils#initDebugTools`
`LeakCanaryUtils.init(true, BuildConfig.IS_ROBO_TEST_BUILD);`
3. Run the test

NOTE: If you don't have any test for the part that you want to check or the memory leaks require different steps to be reproduced then you will have to manually profile the app. Dump the heap and the memory profiler will tell you if you have a memory leak.

More info about [LeakCanary](https://square.github.io/leakcanary/)
