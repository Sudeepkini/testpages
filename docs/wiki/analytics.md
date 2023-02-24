# Analytics
This section describes how to integrate the new analytics SDK. This section is still very much work in progress, so it might be missing some details. Please reach out on `#pd-tech-android` Slack channel for any questions regarding the use of the SDK

## Setup
1. Add `analytics-api` dependency
    ```kotlin
    implementation(projects.analyticsApi)
    ```

2. Inject Analytics
    ```kotlin
    class MyViewModel @Inject constructor(
        ...
        private val analytics: Analytics
        ...
    )
    ```

3. Create event for tracking. You can create either of `FirebaseEvent`, `BrazeEvent`, or `AdjustEvent`
    ```kotlin
    val event = FirebaseEvent(<eventName>) {
      put(key, value)
      putAll(xxx)
    }
    ```

4. Track event
    ```kotlin
    analytics.track(event)
    ```
ℹ️ Quick tip: For testing, the `FakeAnalytics` class provided via `analytics-testing` module should be used, to aid robust testing without any need for mocks


## Experiment analytics

### Dispatching `participated` analytics events

Alongside with every usage of `ab test` and/or `feature flags` from [FwF](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/app-configuration/#feature-toggle-fun-with-flag) feature configuration that is used in the application there is often the need to dispatch `ab_test.participated` and `feature_flag.participated` analytics events to Google Analytics such that every experiment can be evaluated and measured correctly.

In order to abstract the business logic to decide which event should be dispatched from the app, an `ExperimentEventTracker` API is provided, which creates and tracks the correct participated events based on the `VariationInfo` of the feature being evaluated

**Usage:**
* Retrieve the right `VariationInfo` reference for your feature/experiment using the `VariationProvider` or `FeatureToggle` APIs from configs module
* Inject the `ExperimentEventTracker` API into your business logic class
* Invoke the API

For example:
```kotlin
val newCheckoutFlow = configManager.featureToggle.newCheckoutFlow
experimentEventTracker.trackExperiment(newCheckoutFlow)
```
* Since `ExperimentEventTracker` is just a simple interface, this makes it super easy to provide a fake for use in unit tests

ℹ️ Quick tip: Setting the FwF flag to be able to test and receive correct setup for abtest type from FwF. See [here](https://confluence.deliveryhero.com/pages/viewpage.action?spaceKey=GLOBAL&title=Apps+and+Web%3A+Tracking+and+feature+flag+testing)

🚨 Warning: `ParticipatedEventsFactory` is now deprecated and all usages should be migrated to `ExperimentEventTracker`


## Screen Tracking
We provide 2 kinds of APIs to enable efficient screen event tracking

### Implicit API
This API is useful for activities and fragments that want to opt-in for automatic screen event tracking

To enable it for your activity/fragment, simply implement `TrackableScreen` interface from analytics-api

```kotlin
class MyActivity : Activity(), TrackableScreen {
    override val screenParams: ScreenParams
        get() = ScreenParams("screenName", "screenType")
}
```
Please refer to the code documentation on TrackableScreen for more details

🚨 Warning: There's a legacy `TrackableScreen` in tracking module which is now deprecated and all usages should be migrated to TrackableScreen from analytics-api instead.

### Explicit API
This API is useful for cases where you need more flexibility than what the implicit API provides. E.g controlling when the event should be tracked, and/or whether some extra parameters should be included in the event

To use this mode, simply inject `ScreenTracker` and invoke the necessary API as required

For more details, please refer to the code documentation on ScreenTracker interface

## Convention
This section describes a few key guidelines and conventions to follow when using the analytics SDK

1. Events should be created using event factories. The sole purpose of these factories is to create `AnalyticsEvent` which will be tracked at specific points in user journey.

  ```kotlin
  class VendorEventFactory {

    fun createAbcEvent(...): AnalyticsEvent {
      return FirebaseEvent(...) {
        put(key, value)
        putAll(...)
      }
    }

    fun createDefEvent(...): AnalyticsEvent {
      return BrazeEvent(...) {
        put(key, value)
        put(key, value)
        put(key, value)
      }
    }

    ...
  }
  ```
2. Prefer to limit scope of analytics to implementation modules alone. Analytics are usually a result of some action happening on the app. And since these actions are only connected in the implementation modules anyway, we can easily ensure that all analytics happens only inside these modules.
  - For cases where we may need to expose public APIs for feature related tracking purposes, consider exposing simple, focused `*Tracker` interfaces, whose implementation can then involve actual creation and tracking of the analytics events. This also helps to avoid the need to depend on `analytics-api` from another api module. For example
  ```kotlin
  // api module
  interface VendorTracker {
    fun trackVendorClicked(...)
    fun trackVendorInfoOpened(...)
  }

  // implementation
  class VendorTrackerImpl @Inject constructor(
    private val eventFactory: VendorEventFactory,
    private val analytics: Analytics
  ): VendorTracker {
    fun trackVendorClicked(...) {
      ...
      val event = eventFactory.createVendorAbcEvent(...)
      analytics.track(event)
    }

    ...
  }
  ```
