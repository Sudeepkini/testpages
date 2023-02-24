# Brand migration walkthrough

There are some steps from a brand migration that will be repeated for all. This document contains them all and some tips
on how to get them

## Create a new flavor

The first thing we need to do is create a new flavor for the app. To create a new flavor you need to update the `flavors.gradle`
file.

## Signing config

Make sure to get the production keystore and add it to the root of the project.

Create a signing config and add it to the flavor. Use the actual `alias` value from the production keystore, but the `keyPassword` and `storePassword` should be added as project properties. The actual values should be added as secrets in the project's [bitrise workflow editor](https://app.bitrise.io/app/91e92520d8bca806/workflow_editor#!/secrets)

## Updating brand and variant-specific values

**VariantKeys**

Create debug and release instances of the `VariantKeys` class for the new brand in `VariantKeys.kt` file and make sure to add them to `variantKeyMap` with a `{flavor name}+"Debug/Release"` as a key.
Fill values for all the required fields, some of which are shared between different brands, e.g. `GTM container id`, `fwfKey`, `gisApiKey`, `tplMapsApiKey`. For other fields make sure to reach out to the appropriate people.

**IMPORTANT:**
The **debug** Google Maps _API key_ is the same for all the brands because they are all signed with the same debug signing key. But note that it's not enough just copy-paste the key for the maps to start working, since the _API key_ is restricted to application id, so make sure to reach out to someone from the [Location squad](https://confluence.deliveryhero.com/display/GLOBAL/Domain+Location+Experience), to add the debug app id to [this api key](https://console.cloud.google.com/apis/credentials/key/10db77e5-d240-4bef-91f5-e4bc277fccc0?project=dh-gmp-dh-28886).

The release Google Maps _API key_ should be created by the location team specifically for the new brand, based on `SHA1` of the production signing key and the app's id.

**BrandKeys**

Create a new instance of `BrandKeys` class for the new brand in the `BrandKeys.kt` file, fill all the required fields, and add it to the `brandKeyMap` with a flavor name as a key.

## Update google-services.json

We need to get access to the new brands Firebase project, get `google-services.json` and add it on the brand specific source set
If there are more apps in the json, remove them and just keep the two we need `release` and `debug`

**IMPORTANT:**
Make sure that both Android and iOS platforms use the same Firebase project. If not, there are two options:
- only one of the existing projects should be selected
- a completely new project should be created
to use for both platforms. The decision should be made involving the analytics team, just to ensure that we don't lose any important data.

#### Debug flavor doesn't have the same application id

For Pandora apps debug flavors always have the application id as `*release-application-id*.dev`, this might not be true for the brand you're migrating
If that is the case then we need to add a new app in **Firebase** to conform to our application id convention

While testing the app migration you will need to include the original apps `client_info` in `google-services.json`

## Remote config (Firebase)

Make sure not to update/remove anything from the existing remote config on Firebase, since it still might be used by the app before the migration is finished.
To figure out what needs to be added to the remote config you can go through the `RemoteConfigMapper.kt` file and add them one by one to the new project
Make sure to use the default values for the new project

#### Client secrets

`client_secrets` is one of the brand-specific remote configs, which is represented as a JSON object with a country code to a secret pair, for example, that's how the config would look for a multi-country brand:

```
{
   "LA":"aeX6KcwCcwTkcpxV2YuAtwpMJeQy5Vzq4KGyHYhXg8uVZWtV1",
   "KH":"2t5NPZ3YnKub5h9UUyJdgFqZA75dscZuHCMSsrBUZDdrnp2qH",
   "JP":"xvvebcx9ahww0scwkogwc8g8gs8wc4gow8s0sckw0k4s4s03c",
   "DL":"iAeybLCEYeOj/wGlFxAAWcARfMyW8yHIygZhQKLU61qYlg75sUyWiA",
   "SK":"iAeybLCEYeOj/wGlFxAAWcARfMyW8yHIygZhQKLU61qYlg75sUyWiA"
}
```

Make sure you add secrets for all supported by the brand countries there, you can find values of the environment- and platform-specific secrets in the [pd-app-config](https://github.com/deliveryhero/pd-app-config/tree/master/apps/keymaker) repo.

**IMPORTANT:**

There are cases where a new brand is being launched in a country where we already have another Pandora brand (e.g _OnlinePizza_ and _PizzaOnline_ were launched in countries with existing _Foodora_ brands)
In such a case, platform country code may be different from the standard ISO country code, so make sure that these are properly configured for the `client_secrets`.

## Static config (S3)

The static config is country-dependent, so if this is a new country you will need to add this one too. You can do so by opening a PR on this [repo](https://github.com/foodora/static.fd-api.com)

## Global mobile configuration (`/getmobilecountries` endpoint)

This configuration is brand-specific and fetched on the launch of the app. It returns base info about supported countries for the brand. This config is maintained by BE, so there is nothing that should be actively done about it, but it's better to verify that the response is correct.

**IMPORTANT:**

Make sure that the `iso_country_code` field in the response is a 2-letter code (e.g. `TR`, not `TUR`), otherwise it might lead to broken logic in the app.

## Country api configuration (`api/v?/configuration` endpoint)

This configuration is country-specific and fetched when a user's location gets known. It returns all the important info about the country a user currently in. This config is also maintained by BE, so there is nothing that should be actively done about it, except verifying the correctness of the response.


## Braze/Appboy

For Appboy to work we need to add the api key in a values resource file for each variant `*brand*Debug` and `*brand*Release` you can get the content of the xml from the existing brands

#### Braze Push Notification credentials

In order for push notifications to work, we need to provide to our CRM team `Server Key` and `Sender ID` from the Firebase project console.
These credentials can be found in **Settings > Cloud Messaging**. More info in [Braze documentation](https://www.braze.com/docs/developer_guide/platform_integration_guides/android/push_notifications/android/integration/standard_integration/#step-4-set-your-firebase-credentials).

## Adjust

Get an adjust app token and event tokens from the marketing team.
- App token is usually the same for staging and production, it should be added to a respective `BrandModel`
- Event token values belong in `AdjustKeys.kt` (per brand) and `aliases-fat.json` (all brands together) files in the `:app` module

## Shakebugs SDK

Because of the bug in Shakebugs SDK, ProGuard will remove Shakebugs SDK classes and resources from the release build,
if they are not kept explicitly. In order to do so for the new brand, copy the `app/src/netpincer/res/raw/keep_shake.xml`
file to the `app/src/<new_brand>/res/raw/keep_shake.xml` location.

## App languages

When the translations for all supported languages have been added to WebTranslateIt you have to update `locale_config.json` in the [pd-mob-translations](https://github.com/deliveryhero/pd-mob-translations) repository.

## App migration

To implement migration you have to extend `com.deliveryhero.migrations.MigrationsManager.kt` class inside the brand-specific source set

To migrate the app successfully we need to get the last used address and the persisted token (if the app is using oauth2 refresh token)

### Address

In order to get a viable address which would work with our the best way is to use reverse geocoding. So all you would need from the address during the migration is the **lat/lon** coordinates

The address migration logic resides in `com.deliveryhero.migrations.MigrateUserAddressUseCase`, you only need implement an abstract method that reads the existing location.

### Token

For the token to be valid for Pandora services you need to exchange the old token to a new one, for instructions how to do this talk to the appropriate squad which is handling this service
Make sure after you get the new token persist the token in `LocalStorage` and also download the customer data so the user is logged in

At the time of writing this we have two different ways of using authentication, so make sure to save the oauth response
in these two different keys:
- **oauth_token**
- **customer_auth_{country_code}**

### Testing the migration

In order to test the debug migration you have to get the debug `*brand*.keystore` and change the `dev` `signingConfig` from `flavors.gradle`
We have to do this so we can upgrade the debug app to see if the migration is working

It's also important to verify that the migration works properly on the builds signed with the release key, to ensure that obfuscation doesn't break the logic.

**IMPORTANT:**

All of this is temporary and should only be used locally to test the migration, all debug builds after the migration can be signed with `dev.keystore`

## Bundled configs

For every production release, we add bundled configs to the build. In order to get the bundled configs for the new brand you must add the new brand to [pd-configs](https://github.com/deliveryhero/pd-mob-configs) and update version of the configs build in `refresh-config.sh`

**IMPORTANT:**

For the configs service to be able to fetch remote config we need to generate a [service account](https://firebase.google.com/support/guides/service-accounts), this json then should be added to the other jsons on Bitrise:
- Go to the [Code Signing](https://app.bitrise.io/app/91e92520d8bca806/workflow_editor#!/code_signing) tab
- Find and download an archive file with the `BITRISEIO_FIREBASE_KEYS_URL` id in the **GENERIC FILE STORAGE** section
- Unpack the archive and add there the new json (it should follow the same naming format that other brands in the archive have)
- Archive the files and upload again to bitrise (for this you might need to delete the old one)

## Perimeter X

Make sure to talk to someone from [Platform Security squad](https://confluence.deliveryhero.com/display/GLOBAL/Squad+Platform+Security) to enable the new app/brand in Perimeter X

## Digital Asset Links
In order for the app to properly handle [App Links](https://developer.android.com/training/app-links/verify-site-associations), the app-specific digital asset links have to be added to [Static config](https://github.com/deliveryhero/static.fd-api.com/tree/master/s3root/universal-links):
- Go to `Google Play Console -> App Integrity -> App Signing` and find there a JSON snippet under the **Digital Asset Links JSON** header.
- Copy-paste this snippet into `/s3root/universal-links/production/{country_code}/assetlinks.json` and `/s3root/universal-links/staging/{country_code}/assetlinks.json` - if there are no such files, create them. Also, make sure to update the package name for staging.

## Releasing

### Uploading builds to App Center

To be able to automatically upload the new brand builds to App Center:
- Update the `allBrands` and `$brandsIcons` property with the new brand in [fastlane_upload_hockey.rb](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/fastlane/fastlane_upload_hockey.rb)
- Create Alpha/Beta/Live channels for the brand on [App Center](https://appcenter.ms/orgs/hockeyapp-z6mg/applications)
- Add ids of the created channels to [hockerapp]() file (you can find id of a channel in a browser's address bar, for example: _Yemeksepeti-Android-Staging-Alpha_), following the next format:
```
hockeyAppIdFor{Brand}Feature="{alpha channel id from AppCenter}"
hockeyAppIdFor{Brand}Beta="{beta channel id from AppCenter}"
hockeyAppIdFor{Brand}Release="{live channel id from AppCenter}"
```

### Uploading builds to Google Play

To be able to automatically upload the new brand builds to **alpha/beta** channels on Google Play, the [Fastfile](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/fastlane/Fastfile) script should be updated:
- Add two new lanes `release_*new brand*_beta` and `releae_*new brand*_alpha` and also update the corresponding lanes `release_all_beta` and `release_all_alpha` to call these new lanes
