# Mobile Service Provider Integration

Our codebase is set up in such a way as to support integrating multiple mobile service providers (SP). Currently, we support 2 SPs:

1. Google Mobile Services (GMS)
2. Huawei Mobile Services (HMS)

## Design
As a result of the nature of this integration, all SP aware dependencies MUST be removed from all modules and replaced with SP agnostic abstractions.
These abstractions are exposed like typical -api modules and shall be consumed as such. The implementation for these APIs however, is a bit more tricky,
since we needed them to be exclusive. A GMS build should not know anything about HMS dependencies, and vice versa

After considering several factors, we came up with 2 potential approaches to integrating these services

1. Using individual modules for each SP implementation (e.g analytics-gms, analytics-hms). However, this resulted in way too many modules, and turned out to not be flexible enough in terms of code sharing and/or fallback in cases where we don't necessarily have implementation for an SP
2. Making app module SP aware. While this worked fine, it required invalidating pretty much all the tools we have because tasks like `assembleFoodpandaDebug` simply don't exist anymore. We'd have to change to `assembleFoodpandaGmsDebug` for example, which easily complicated the tooling support. This also multiplied the number of Gradle tasks as it's now combinations of brand and SPs

Our final solution is a hybrid of both of these solutions, in terms of the benefits.

1. Take advantage of Android flavors to keep code for all providers in a single module. This enables code sharing and easy fallback strategy when necessary
2. Avoid invalidating all existing tools and automations by maintaining the task structure, and instead, using a Gradle property (`com.deliveryhero.service.provider`) to decide which resolution strategy to use for the SP modules. This property can then be overridden accordingly to make a new build for the specified SP

**Note**: For a reference implementation, please take a look at `analytics-sp` and `analytics-sp-api` modules

## Convention
- SP modules MUST be suffixed with `-sp`. E.g `analytics-sp`, `analytics-sp-api`
- Keep SP modules (both api and implementation) clean from any b2c specific logic.
  - SP modules should be exclusively providing high level SP logic. Every feature specifics should exist in the actual feature module, and only taking advantage of those high level APIs from SP.
- This means SP modules (api/implementation) CANNOT depend on other b2c modules
- Every other existing module conventions continue to apply to SP modules
- Keep dependencies as lean as possible
  - Avoid any dependency in API modules if possible
  - Prefer to add only the very dependencies that are absolutely required in implementation modules

## Toggle features remotely (FWF, Firebase Remote config) for different Service Providers 
In order to be able to turn on/off features for different service providers there is custom attribute `mobile_service_provider` available on FWF and soon (early October 2022) will be added in Firebase Remote Config.
Available values for this custom attribute are `hms` (Feature to be enabled for Huawei AppGallery) and `gms` (Feature to be enabled for Google PlayStore).
