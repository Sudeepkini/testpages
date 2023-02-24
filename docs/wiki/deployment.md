# Travis (Development)
[![Build Status](https://travis-ci.com/deliveryhero/pd-mob-b2c-android.svg?token=dVY4piTgUD2ys38vXayd&branch=development)](https://travis-ci.com/deliveryhero/pd-mob-b2c-android)

# Bitrise
[![Build Status](https://app.bitrise.io/app/91e92520d8bca806/status.svg?token=ibBAUEFHyPEcTDyhQyZZOg)](https://app.bitrise.io/app/91e92520d8bca806)

Uploading to Hockey via Fastlanes
=
1. From console go to the branch from where you want to upload builds
2. The fastlane command is ```upload_hockey```
3. To run the command type ```fastlane upload_hockey t:rc-all```. All variations for the type ```t:xxx``` are specified below

   |  ** Type ```t:xxxx``` **  |                      ** Result **                           |
   |---------------------------|-------------------------------------------------------------|
   | rc-all                    | Uploads production and debug(proguarded) version to hockey  |
   | rc-debug                  | Uploads debug(proguarded) version to hockey                 |
   | rc-live                   | Uploads production version to hockey. Only possible through Bitrise workflow `Upload-All-Production-on-HockeyApp`                        |
   | feature                   | Uploads debug version                                       |

4. Also you can specify which brands you want to upload. Adding the parameter ```b:``` with comma separated value. Example ````b:foodora,foodpanda,onlinepizza,pizzaonline```


Uploading Release to Beta/Alpha on Google Store via Fastlanes
=
1. Go to your release branch and type ```fastlane release_{brandName}_beta``` or ```fastlane release_{brandName}_alpha```
2. You can also upload all at once. ```fastlane release_all_beta``` or ```fastlane release_all_alpha```


Uploading to Hockey via ```./uploadToHockey``` script
=
1. From console go to the root of the priject
2. Execute ```./uploadToHockey``` <-- this is a shell-script (Don't try to run gradle build with that name)
3. Follow the choices/instructions
4. Hockey App Id will be selected based on the following criteria:

   | **Build Type** | **Branch Name** | **API Environment** |    **Hockey App**       |
   |----------------|-----------------|---------------------|-------------------------|
   | Release        | *               | *                   | Foodora Live            |
   | Debug          | *               | crowdtest           | Foodora Crowd Test      |
   | Debug          | development     | staging or qa1-qa4  | Foodora Beta            |
   | Debug          | release/*       | staging or qa1-qa4  | Foodora Beta            |
   | Debug          | other branches  | staging or qa1-qa4  | Foodora Alpha (Feature) |

App Updates & Force Update
=
In order to trigger a force update for the app do the following:
1. Release a new fix version to Google Play.
2. Login to [Firebase](https://console.firebase.google.com) console and navigate to [Remote Config](https://console.firebase.google.com/project/android-foodora/config).
3. Locate the key named `app_force_update_version` which is a JSON string.
4. Add a new entry in the JSON to indicate the versions that should receive a force-update.
5. Your JSOIN should like the following:
```json
{
  "build_numbers": [
    {"from": 527, "to": 529},
    {"from": 100, "to": 400},
    {"from": 549, "to": 549, "translation_key": "NEXTGEN_FORCE_UPDATE_TEST", "alternative_message": "force update test message"}
  ]
}
```
6. Fields are as follows:

  | **Field**                        | **Description**                                                          |
  |----------------------------------|--------------------------------------------------------------------------|
  | from *(Required)*                | Min version number to receive force update                               |
  | to  *(Required)*                 | Max version number to receive force update                               |
  | translation_key *(Optional)*     | String key to lookup in translation to display update message            |
  | alternative_message *(Optional)* | if no `translation_key` and no translations found this will be displayed |


Building APKs locally
=

To create builds locally, please go through this [link](https://developer.android.com/training/basics/firstapp/running-app)

## to overwrite API declarations
- run task :changed-module-api:apiDump
