# Additional info

## Important!!!

* All Activities should have this option `android:screenOrientation="portrait"`. The apps work only in portrait mode, we dont support landscape.
* Glide lib is used for image processing. Please always use optimized url which accepts `width` and `height` as query params in the url (DH image service works like this). For providing optimized url based on network quality and device screen width, please use `ImageBuilder` abstract class and use one of its methods.
* Never use `android:includeFontPadding` in `TextView` as this cuts the text in some languages like Thai. Reference:
https://github.com/deliveryhero/pd-mob-b2c-android/pull/3210
* Always add `@SerializedName` with expected field names to API models, otherwise, the fields will be obfuscated and this will break the parsing logic.
* Please always make use of `DeviceUtils` class for Animations form where you can use `shouldDoBasicAnimations()` and `shouldDoAdvancedAnimations()` methods. Please avoid Animations for low end devices by using this method to determine if should run animations or not.

## Add _Adjust_ information provided by the marketing team for a new brand in 5 easy steps:

1. Add the _Adjust_ secret in the correct flavour like this in _BrandKeys.kt_:<br />
`adjustSecret = "new long[]{<values here are given by the marketing team>}"`
1. Add the _Adjust_ token in the _VariantKeys.kt_ file
1. Copy the _AdjustKeys.kt_ file to the new flavour folder and replace all values.
1. So easy you don't need 5 steps.

# Parsing phone numbers

If you need to parse phone numbers you can use the interface `com.deliveryhero.commons.PhoneNumberParser` available in the _commons_ module from _metalcore_.

In our case, PhoneNumberParser is an abstraction over _libphonenumber_.
The original library from Google and its Java implementation is not optimized for Android, for that reason the version that we're using is a fork from the Google's project.

* Google version: https://github.com/google/libphonenumber
* Android optimized version: https://github.com/MichaelRocks/libphonenumber-android

Please refer to the _README_ of both projects for the reasons on why to use the Android optimized version.

# Whetstone Experiences Sharing
With Anvil and Whetstone, we can use Dagger in a simpler and easier way. Anvil simplifies the part of `@Component` and `@Module`, and Whetstone extends it to make our own processor help us work with Android components easier.

Another benefit Anvil brings is reducing our project compile-time that it generates Dagger factories using [KSP (Kotlin Symbol Processing)](https://kotlinlang.org/docs/ksp-overview.html) instead of [kapt (Kotlin Annotation Processing Tool)](https://kotlinlang.org/docs/kapt.html). (Find more detail here: [Accelerated Kotlin build times with KSP 1.0](https://android-developers.googleblog.com/2021/09/accelerated-kotlin-build-times-with.html), [Why KSP](https://kotlinlang.org/docs/ksp-why-ksp.html#top))

However, there's a limitation of using `internal` modifier:
* Whetstone is a Kotlin Compiler Plugin and therefore, we can’t share internal dependencies between a monolithic graph (we use a similar approach from Hilt). That is, we have to remove the `internal` modifier if you want to expose classes to `ApplicationScope`. (Refer to [thread](https://deliveryhero.slack.com/archives/C02DHS7KN6Q/p1633095441002300?thread_ts=1633094641.000500&cid=C02DHS7KN6Q), [thread](https://deliveryhero.slack.com/archives/GP93NLRB4/p1643177792008500?thread_ts=1643172507.006400&cid=GP93NLRB4).)

### If you're migrating Dagger to Whetstone,
* Do your initializations in a separate class that extends `Initializer` class, such as initialize `GtmTrackerRegisterer` or any in your own component init block. (Refer `VerticalsInitializer`)
* Dependencies flow downwards (and never upwards). So for scope, if class A is in `ApplicationScope` and depends on class B then class B must be either on the same level or higher in the graph hierarchy. (More detail [here](https://github.com/deliveryhero/pd-mob-b2c-android/pull/12695),  [lifecycle diagram](https://github.com/deliveryhero/whetstone#component-lifecycle))
* MVVM is easy in Anvil(due to `ViewModel` scope) but a bit different in MVP.
* If you inherit classes and the base class has `@inject` it will sometimes show issues on compile-time sometimes it does not.
* Refer to these PRs that did migrations. [POST-4977](https://github.com/deliveryhero/pd-mob-b2c-android/pull/14005/files), [COR-2663](https://github.com/deliveryhero/pd-mob-b2c-android/pull/12695), [COR-2423](https://github.com/deliveryhero/pd-mob-b2c-android/pull/12265/files), [SMNU-1936](https://github.com/deliveryhero/pd-mob-b2c-android/pull/12848/commits/52b1536eb80b84d0c4d3a5feb9a04e0de4a53639)

### Last but it's worth checking these again in your code!
* Must call `Whetstone.inject(this)` before `super.onCreate()` in Activities or the app will crash!
  * `Whetstone.inject(this)` helps us do field injections which are “nullable” most of the time with `lateinit` so if we don't call it explicitly the app will crash while we try to call this field.
  * If present it in base activity then don't add in your activity extending that. Otherwise, multiple instances will get created for each dependency.
  * If you put it after `super.onCreate()` you'll get crashes when the activity recreates itself having fragments, more description in [PR](https://github.com/deliveryhero/pd-mob-b2c-android/pull/13289).
* `@Reusable` does not guarantee that the object will actually be reused. So, if your object holds a state that you wish to share, only `@Singleton` can do that.
* Not to overuse `@Singleton`, since usually it will be installed into a global AppScope. Instead, you can narrow the scope down to `@ViewModel` or `@ActivityScope`.

Find the complete sharing [here](https://deliveryhero.slack.com/archives/GP93NLRB4/p1643172507006400).
