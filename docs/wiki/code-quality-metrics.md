## What is this documentation for?
This document is a series of guides we have written for the Code Quality Dashboard that we use to track and improve our overall code quality of the project.
These guides are mostly targeted to people who want to know how the internals of the framework works.

## How do we gather code quality metrics?
To gather the code quality metrics we use a bunch of static analysis and lint tools.
We wrote different scripts to run these tools and then we parse the outputs to a different format in order to upload and save the data on BigQuery.
The main workflow which runs periodically (every Sunday at 15:00 CET) is **Collect-And-Upload-Code-Quality-Metric**.

**Note:** For definitions and impact of each metrics you can read more about it in the [*Glossary*](https://datastudio.google.com/u/0/reporting/4dbd9545-c270-490c-83f8-959ad086b161/page/p_amnuv36wyc) page of the dashboard.
**Note:** This workflow only exists on the Android Bitrise app. It updates the metrics for both projects.
### Code coverage
**Android:** We are using [`jacoco`](https://www.eclemma.org/jacoco/) to run code coverage for the whole project. We're using a custom gradle task to run `jacoco` for the whole project and parse the report before we send it to BigQuery. More can be found in this file [`jacoco-bq-reports.gradle.kts`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/config/jacoco-bq-reports.gradle.kts).
We're getting the coverage based on `instructions` as seen on the `jacoco` report.

**iOS:** We are using [xcov](https://docs.fastlane.tools/actions/xcov/)

### Issues
**Android:** Currently we're using [`Detekt`](https://detekt.dev/) to collect issues. The only issues we collect for the code quality framework are `style` and `formatting` issues. We have chosen only this class of issues for our first POC.
The relevant script is [`generate_detekt_report.sh`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/scripts/code-quality-framework/generate_detekt_report.sh).

**iOS:** We're using [`Swiftlint`](https://realm.github.io/SwiftLint/) and we're collecting the same issues which are reported on every PR.
You can check `swiftlint.sh` to see how we execute `Swiftlint`.

### Complexity
**Android:** For complexity we use `Detekt`. We're getting the values from it's html report. Check `generate_detekt_report.sh` for more information.

**iOS:** We use `Swiftlint` to gather complexity. You can check [`swiftlint.sh`](https://github.com/deliveryhero/pd-mob-b2c-ios/blob/development/CodeQualityFramework/swiftlint.sh) to see how we execute `Swiftlint`.

### Duplication
**Android:** To find duplicated code we use a tool from `PMD` called [`C(opy)P(aste)D(etector)`](https://pmd.github.io/latest/pmd_userdocs_cpd.html) which can find duplicated tokens within the whole project. Check [`scripts/cpd-copy-paste-detection.sh`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/scripts/cpd-copy-paste-detection.sh) to see the configuration and how we run the script.

**iOS**: We use the same tool as in Android. You can check [`detect-duplicate-code.sh`](https://github.com/deliveryhero/pd-mob-b2c-ios/blob/development/CodeQualityFramework/detect-duplicate-code.sh) to see how we run the tool.

**Note:** In both platforms we're using a minimum of 100 tokens being repeated as the definition of duplicate code. This is open to change as we fine tune the tool to better fit our needs.
**Note:** For Android we're ignoring generated and test code. In both these case we're fine with duplication happening in these source sets. On iOS we don't have any exclusion rules as of yet.

### Pandora issues/Initiatives
**Android:** These initiatives are collected through different means. Although most of these issues are coming from `Android Lint`. You can find how these issues are collected from [`QualityDashboardTask`](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/build-logic/src/main/kotlin/com/deliveryhero/convention/android/QualityDashboardTask.kt). Other issues include **deprecations** which are collected using `Detekt` and missing `-api` modules which are found and reported by `detect_module_structure.sh`.

**iOS:** On iOS we currently have only one issue/initiative, that is the deprecations. We read deprecations from `XCode` log dump. More information [`CodeQualityFramework/deprecation.rb`](https://github.com/deliveryhero/pd-mob-b2c-ios/blob/development/CodeQualityFramework/deprecation.rb).

#### How to add new initiatives?

Almost all of our initiatives are currently being found by using Android Lint and custom lint rules we define on `lint-rules` module. Adding new initiatives will require these steps:
1. Create an Android Lint rule which finds the issues you are looking for
   1. Rule id should follow this format: `pandora.{{issue group}}`.{{issue name}}`
2. Add rule id to list of Pandora issues in `pandora-issues.properties`
3. Add a description to the issue in `pandora-ta-results.mobinfra.pandora-issue-description` table on Big Query

After following these steps make sure to run the Bitrise workflow to see your issue in the dashboard.
