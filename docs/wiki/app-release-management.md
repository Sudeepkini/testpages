# Release process
* Updated release proccess can be found [here](https://confluence.deliveryhero.com/pages/viewpage.action?spaceKey=GLOBAL&title=Mobile+Release+Process)
* Mobile apps have **bi-weekly** release train for both Android and iOS. Release schedule dates you can find it [here](https://confluence.deliveryhero.com/display/GLOBAL/Mobile+Releases+Schedule+2021)
* **Code freeze** happens on every second Friday end of the day when release branch is created out of development. First builds must be shared Friday end of day on #pd-mobile-releases Slack channel.
* Regression testing happens **from Monday until Thursday** performed on the beta staging builds and all bugfixes should be merged until Thursday end of day and Friday will be left for teams to verify last builds with all fixes are stable.
* During 5 days regression **new builds should be provided every day** at the end if there were fixes merged to the release branch.
* After all squads are ready on Friday **Public Beta version** will be uploaded on PlayStore.
* On Monday the results will be checked for crashes and non-fatal issues and if needed additional fixes will be provided before starting **Production phased rollout** at the end of the day.
* All RMs should get all `Admin` access on all PlayStore accounts in order to be able to release new apps and invite new users when requested to our team. **Access to the owner accounts** has the [GCS team](https://confluence.deliveryhero.com/display/SRE/DH+Mobile+Application+Credential+Governance). If there is need to perform any action only by the owner account please submit general request [here](https://jira.deliveryhero.com/plugins/servlet/desk/portal/38) for the GCS team specifying for which accounts is needed the owner access from this [list](https://docs.google.com/spreadsheets/d/1CKCHalc7hoTsZESxV8yu80vERKcuKABYpYVJ3OLS05U/edit#gid=126479778).

# PlayStore Console Tips
* Delivery Hero is part of the Google Play partnership program. Our apps are prioritized in the review queue in order to help us minimize any disruption to the business which means our apps should be reviewed in less than 72 hours.
* In case of any need to contact Play Store support team for issues related to our apps please always write an email to `play-bd@google.com`.
* **Always use staged rollout**. Never ever release the apps to 100% immediately regardless of how many changes are there and how low the risk would be calculated.
* If we have requirements to make some changes on PlayStore that are expected to be live on a specific date and time please prepare everything in advance (please note the 72 hours for review mentioned above) and always use [Managed Publishing](https://support.google.com/googleplay/android-developer/answer/9859654?hl=en) - ON so we can control when to push changes live after they are approved by PlayStore.
    * New app build: Please align with the stakeholders to have the app version to be live with staged rollout in a previous release such that it can be tested. Of course here it makes difference based on the requirements if we are launching new app migration or country launch in existing App.
    * Releasing new app build, changing country availability, content, languages, screenshots...: Requires full review from PlayStore and these changes will not be immediate.
    * Increasing rolout percentages doesn't require full review from PlayStore side.

## Pre-Release
### Code Freeze and Initial regression builds
* Before starting the following process make sure you have **access on Google Play Console** for all brands. If not please request this from Mobile Infra squad on Slack channel ##pd-squad-app-performance-reliability.
* On the day of the code freeze, trigger the `Create-Release` workflow on Bitrise using `development` branch with passing additional custom environment variables in the Advanced page specifying the Release version: Key -> `VERSION` and Value -> `The new release version in format XX.YY.Z, e.g. 21.24.0`. This will create release branch with the format example `release/android/21.24.0`. Please note that this automation step through Bitrise should be skipped when we need to create the first version for new year, in this case please connect with the Release Engineer for support.
* Once the previous step is successful, Next steps are to provide builds for Regression Test for all brands. Trigger the workflow `Generate-RCs-And-Slack-Share` on Bitrise manually for Mob B2C Android app against the newly created release branch. This will automatically build staging and production versions of the apps and share message in `#pd-mobile-releases` Slack channel.
* Change the topic channel of `#pd-mobile-releases` for Android Release Manager with the version and your name tagged `Android (21.24.0) - @Name`
* Assign yourself as Release Engineer in the [confluence page](https://confluence.deliveryhero.com/display/GLOBAL/Mobile+Releases+Schedule+2021) for the specifc release version.

### Regression time (Monday - Friday)
* During the regression week as all engineers are able to merge PRs into the release branch the RM should monitor the amount and type of PRs merging into the release branch using this [dashboard](https://datastudio.google.com/u/1/reporting/9e309883-4105-4191-b011-76eed8f65074/page/Ct8XC). If you notice that single squad contributes with many fixes please review the tickets and challange why is that happening.
* Provide _new builds every day_ at the end of the day in case there were some fixes merged by triggering the workflow `Generate-RCs-And-Slack-Share` on Bitrise. All builds will be automatically shared in the `#pd-mobile-releases` Slack channel.


### Post-Regression

* How the RM will know if regression is complete in order to proceed with Production/Beta release?
1. TestRail dashboard needs to have 0 "Needs Update" test cases.
2. Jira Regression board should have 0 Blockers and Critical bug tickets.

* After all teams go through the last testing session on Friday and if all teams are ready on Friday EOD Release build should be uploaded on Google Play Console to **Public Beta** tracks. Trigger `Release-All-Betas-on-Google-Play` workflow on Bitrise to automatically release all brands on Play Store Public Beta.
* Provide new RC builds by triggering the workflow Generate-RCs-And-Slack-Share on Bitrise.

[//]: # (TODO: PED-151 update this documentation once the release lane documentation is finalized - https://docs.google.com/document/d/1ewJE12W81wY6jNrqVt06uy7S5zWW-kDDVy_1P5xUAmM/edit?usp=sharing)
To allow some features in the app to be only accessible in Beta builds we pass params `isBeta:true` to the workflow as shown this [screenshot](https://user-images.githubusercontent.com/8607770/119530446-4c18ef00-bd83-11eb-8750-fe6f433be403.png)

**IMPORTANT Note** This step is optional and only needed when there are required features/tools that only need to be tested on Beta channel. Please keep in mind that even if the betas were released with `true` flag, on the day of the release the flag will, by default, set as `false`.


* Please share a link from Firebase Crashlytics in `#pd-tech-android` Slack channel with `@here` notifying the whole team about the Beta results for crashes right after submitting the apps to Public Beta. Please construct the link by replacing `{VERSION_NAME}` with the version name and `{BUILD_NUMBER}` with the build number from the Beta Release. This step is important to be completed on Friday such that all teams that are starting earlier on Monday from different time zones to be able to monitor for the crashes.
`https://console.firebase.google.com/u/0/project/android-foodora/crashlytics/app/android:com.global.foodpanda.android/issues?time=last-seven-days&versions={VERSION_NAME}%20({BUILD_NUMBER})&state=open&type=crash&tag=all`

Example, https://console.firebase.google.com/u/0/project/android-foodora/crashlytics/app/android:com.global.foodpanda.android/issues?time=last-seven-days&state=all&tag=all&versions=22.4.0%20(212218147)&types=crash


## Release

* Review the results on Firebase Crashlytics on Monday and pair with the Release Engineer to ping specific teams in case of any new/spiking crashes or non-fatal issues. Fixes for these issues should be done on Monday and release to production the latest build.

* Provide new builds in case there were new fixes from the Public Beta merged, by triggering the workflow `Generate-RCs-And-Slack-Share` on Bitrise.

* If there are new changes on Monday after Public Beta release or Friday's Beta version was released with `isBeta:true` then please run `Release-All-Betas-on-Google-Play` workflow on Bitrise to automatically release all brands on Play Store Public Beta.

* After all critical issues are fixed and all squads are ready, promoting Beta builds to **Production** should be performed as follows:
    * Go to Google Play Console > Open Testing > Locate the latest Beta release and click Promote release > Production.
    * Under `Release Details` > Verify that the `Release name` is correct, e.g. `21.24.0`
    * At `Release notes` Choose "Copy from a previous release" for adding release notes. Here, choose the last production build to copy the release notes from > Select "Copy notes".
    * Click "Save" next to the disabled "Review release" button.
    * Verify release name > Note down the release version number (**you'll need this later) > Select "Review release".
    * Under the "Staged roll-out" section, set the roll-out the initial percentage as: 0.5% Foodpanda APAC, 5% Damejidlo and Foodpanda EU, 10% Foodora NO, SE, FI, Mjam.
    * Finally select "Start rollout to Production" > Press "Rollout" on the confirmation dialog.
* Announce in the Slack channel #pd-mobile-releases about the rollout with an included percentage per brand, version name and release version number (**the one you noted down earlier). Copy the message format from the last release.
* Fill the table in [Mobile Releases Schedule](https://confluence.deliveryhero.com/display/GLOBAL/Mobile+Releases+Schedule+2022) with the correct build number, release date and add your name under the "Release Engineer" column.
* Please pair with the Release Engineer to help you to create tag on Github for this release. On b2c release new tag on Github from release branch and name in this example should be `21.24.0` and if possible provide small description of which features were included.


## Post-Release
* **Monitor daily** for crashes and non-fatals in Firebase Crashlytics.
* **Monitor daily** for conversion rate all CVRs on this [Data Studio Dashboard](https://datastudio.google.com/u/1/reporting/1_wL9cBKb9IRxyOmq8jNAKPg-m7WGX5r2/page/v7fW). Filters per countries can be applied to be able to check for all different brand apps in Pandora. Also please filter the current release version and the previous one to be able to compare the CVRs differences to identify any drop.

* Second day (Tuesday) if everything is stable rollout can be increased to 1% Foodpanda APAC, 10% Damejidlo and Foodpanda EU, 20% Foodora NO, SE, FI, Mjam.
* Third day (Wednesday) if everything is stable rollout can be increased to 5% Foodpanda APAC, 20% Damejidlo and Foodpanda EU, 50% Foodora NO, SE, FI, Mjam.
* Fourth day (Thursday) if everything is stable rollout can be increased to 10% Foodpanda APAC, 50% Damejidlo and Foodpanda EU, 99% Foodora NO, SE, FI, Mjam.
* Fifth day (Friday) if everything is stable rollout can be increased to 20% Foodpanda APAC, 99% Damejidlo and Foodpanda EU, 99% Foodora NO, SE, FI, Mjam.
* Sixth day (Monday) if everything is stable rollout can be increased to 50% Foodpanda APAC, 99% Damejidlo and Foodpanda EU, 99% Foodora NO, SE, FI, Mjam.
* Seventh day (Tuesday) if everything is stable rollout can be increased to 99% Foodpanda APAC, 100% Damejidlo and Foodpanda EU, 100% Foodora NO, SE, FI, Mjam.
* Eight day (Wednesday) if everything is stable rollout can be increased to 100% Foodpanda APAC, 100% Damejidlo and Foodpanda EU, 100% Foodora NO, SE, FI, Mjam.


### Managing Production crashes

We use the phased release as a safety net for severe errors that were not identified during the regression. When we encounter new and spiking crashes on production there are 2 ways for prioritization:

**Emergency crashes:**
Every crash that makes daily crash-free users below 99.0%, the fix should be prioritized immediately as a hotfix. Process for emergency crashes:
1. Release manager should **stop the phase rollout and start a hotfix release**.
2. Release manager should create Jira bug tickets for emergency crashes (with the “emergency-crash” label) in the related squad board.
3. Release manager escalates to the relative team in order to have them fix the crash.

**Non-emergency crashes:**
**Top 3 crashes** for the last fully released version, fix can be prioritized for the next release.
Process for non-emergency crashed:
1. Release manager should create Jira bug tickets for non-emergency crashes (with the “crash” label) in the related squad board.
2. Release manager should link the ticket into the Firebase dashboard using the note feature for each crash.
3. PM creates the JIRA dashboard to monitor those crash tickets.


## Regression report creation:
**With every release, we need to generate a report that summarize:**

* **Quality: Number of issues reported and fixed on the release.**
* **Time: Number of "Needs update" Testrail scripts at the end of regression time (Friday) - Number of still open Jira tickets at the end of the regression.**

For that: we have this [Report template](https://docs.google.com/spreadsheets/d/1qoDLUIbV90CIY0lfTTldYci6RHn9e73wvOWzluntSE0/edit#gid=465870102) that you need to duplicate and update for each release.

This report has been automated into this Data Studio [dashboard](https://datastudio.google.com/u/1/reporting/9e309883-4105-4191-b011-76eed8f65074/page/Ct8XC)



## Hotfix steps in case the release is not stable with different reasons like new/spiking crashes or conversion rate issues
* Trigger the `Create-Hotfix` workflow on Bitrise using `development` branch with passing 2 additional custom environment variables in the Advanced page specifying the Hotfix version: Key -> `VERSION` and Value -> `The new release version in format XX.YY.Z, e.g. 21.24.1` and base Tag from which the hotfix branch should be based on: Key -> `TAG` and Value -> `Tag name e.g. 21.24.0`. This will create release branch with the format example `hotfix/android/21.24.1`. Please note that this automation step through Bitrise is only available for release tags from 21.24.0 and above.
* Follow previous release steps to provide production and staging builds for smoke test for the resolved issues.
* After releasing the hotfix builds to production, fill the table in Mobile Releases Schedule with the correct build number, releasing new tag on Github with tag Name (e.g. `21.24.1`).
* In case of multiple hotfixes repeat the same steps with increasing patch version (e.g. second hotfix will have version 21.24.2).

### Hotfix steps for P1 Critical Incidents related to mobile apps
* Create a hotfix branch from previously, fully released and stable version (see hotfix steps above).
* Upload on the stores reverted version from the hotfix branch created in point 1, with higher build number and manual release action. This will help to run stores review process and investigation for the root cause in parallel.
* Incident Commander to nominate few QA specialists to perform smoke test on the new reverted production builds by also updating the app on top of the new buggy released version. We need to identify backward compatibility issues as this is reverted version. TODO: QA and TA chapters to define clear critical test suite that needs to be executed.
* Decision to release this version to production will be made based on the criticality and the investigation progress. Final call should be taken by the incident commander assigned on the incident.

# Further Readings
* https://trunkbaseddevelopment.com/
* https://github.com/deliveryhero/pd-box/issues/1387
