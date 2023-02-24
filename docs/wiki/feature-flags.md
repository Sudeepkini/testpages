## Introduction

We use feature flags on our project for experimentation and to safeguard some features. For more information on what kind of configurations we have, check the [Configurations](app-configuration.md) page.
In this page we will mostly be talking about [FwF](https://management.fwf.deliveryhero.net/) and [`FeatureToggle`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/configs/src/main/java/com/deliveryhero/configs/featuretoggle/FeatureToggle.kt).

## How to add a new feature flag to the project?

All feature flags should be decentralized and that means that if you're adding a new feature flag it needs to be inside your implementation module. 
We achieve this by adding `extensions methods` to `FeatureToggle` class or by creating a new class which has a dependency on `FeatureToggle` and then querying these flags using the flag name.
`FeatureToggle` is a wrapper around a `Map` so you can use the flag name as key and get back a [`VariationInfo`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/configs/src/main/java/com/deliveryhero/configs/featuretoggle/VariationInfo.kt). Using this object you can understand whether the feature flag is on or off.

### What can I do if I need to use my feature flag outside my implementation project?

There are many cases where a feature flag would need to be available to other modules, in this case we have some simple rules.

- Make sure that you absolutely need to expose your feature flag to other modules
- Provide an abstraction of your feature flag (eg: Don't expose `VariationInfo` instead provide a `StateModel` or `Boolean`)
- Offer a callback when your component is shown or a trigger to analytics needs to be invoked

By doing this we make sure to keep our `-api` modules as clean as possible, while also allowing some flexibility for a feature flag to be owned by a single project
