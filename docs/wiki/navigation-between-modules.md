## Overview

**Important Note:** This style of navigation is possible because of **Whetstone**, our in house DI library. See [this](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/dependency-injection/) segment of the wiki if you're not familiar with **Whetstone**.

The aim of this pattern is helping teams properly work with navigation in a clean consistent way.

To implement, a Navigator interface/contract needs to be defined. This contract defines how the team wants their activity to be interacted with. This way the consumer has no information about the implementation of the navigator and relies exclusively on the inverted dependency the interface provides, an advantage this provides is a consistent api surface area in the _feature-api_ module that can be `validated`.


**Interesting note:**
We run pr checks on _feature-api_ modules. This helps us maintain and validate the stability of feature apis.

**Dependency Architecture**

<img width="1116" alt="Screenshot 2022-04-12 at 14 40 18" src="https://user-images.githubusercontent.com/678974/162964443-03184fa9-63f0-411b-bba0-f470093df9cb.png">


## How to use?
* To provide navigation access to XActivity from :feature-x module, for example.

an API like below, should be defined in the :feature-x-api module. (Same applies to :feature-y, but is skipped for brevity)
```kotlin
interface FeatureXNavigator {
    fun newIntent(context: Context, params: XyzParams): Intent
}
```

```kotlin
@Parcelize
class XyzParams(
    val myString: String,
    val myInt: Int,
    val myParcelable: Parcelable
) : Parcelable
```

an implementation like below should be defined in the :feature-x module. the :feature-x requires whetstone be properly setup.
```kotlin
@ContributesBinding(ApplicationScope::class)
class FeatureXNavigatorImpl @Inject constructor(): FeatureXNavigator {
    override fun newIntent(
              context: Context,
              params: XyzParams
    ): Intent = Intent(context, XActivity::class.java) // Modify intent as needed
}
```

* To navigate to XActivity from :feature-y module, add a dependency on the :feature-x-api module and @Inject a FeatureXNavigator instance where you need it. FYI: Whetstone generates the required bindings to make this work.

```kotlin
@Inject lateinit var navigator: FeatureXNavigator

override fun onCreate(savedInstanceState: Bundle?) {
    Whetstone.inject(this)
    super.onCreate(savedInstanceState)
    val extra: XyzParams = ...
    val xIntent = navigator.newIntent(this, params: extra)
    startActivity(xIntent) //Launch XActivity
}
```

**Important note:**

This navigation pattern is specifically for cross-feature module navigation.
Internal navigation within a feature module is out of scope.
