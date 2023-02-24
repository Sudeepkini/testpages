## How to update GTM or DTM containers?

The below containers are used to dynamically manage tags to grasp all that marketing data.
- GTM or Google Tag Manager to manage devices with play services
- DTM or Dynamic Tag Manager to manage devices with Huawei services 

These tags can be downloaded as a JSON file from the portal by marketing team. 

The files currently reside in the `pd-mob-b2c-android` project at `analytics-sp` module in `{productFlavour}/assets/containers/{containerFile.json}`

The container files are maintained in a separate repository `pd-mobile-gtm-containers` and is synced with the `pd-mob-b2c-android` repository through Bitrise workflows.


#### Steps to update GTM or DTM containers
- The GTM or DTM json files needs to be added in the appropriate directories in `pd-mobile-gtm-containers` under
  - GTM files with flavour `[debug | release]` needs to be added at `https://github.com/deliveryhero/pd-mobile-gtm-containers/tree/main/android/_ALL_/`
  - DTM files with flavour `[debug | release]` needs to be added at `https://github.com/deliveryhero/pd-mobile-gtm-containers/tree/main/android/hms/`
- Run Bitrise workflow `Update-GTM-Containers-Full` with `Mob B2C Android` project for `development` or any other branch it needs to be merged, by clicking on `Start/schedule a build`
- Once this workflow finishes successfully, **That's it. IT'S DONE!**


### Notes
 - The name of the file should never be changed and should be used as it is provided by the marketing/data team. 
 - This `Update-GTM-Containers-Full` workflow is automatically run as part of generating RCs in the code freeze week.
