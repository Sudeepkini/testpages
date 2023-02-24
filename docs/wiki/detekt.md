# Detekt in b2c
You can find our main configuration file for Detekt through [this link](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/config/quality/detekt/detekt.yml).

### Stack
We use [Danger](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/Dangerfile) to report
newly introduced code quality problems on newly created PRs. This reflects our [main config file](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/config/quality/detekt/detekt.yml).

### Setup
We're setting up Detekt in our [Convention Plugin](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/build-logic/src/main/kotlin/com/deliveryhero/convention/ConventionPlugin.kt).  
We're also using this [gradle task](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/config/reports.gradle.kts)
to merge generated report from hundreds of modules that we have into one single file. Because each module generates
its own report file and at the end we need one singular file for Danger to be able to report newly introduced warnings.

### Code Quality Dashboard
We have a [second configuration file](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/config/detekt-complexity.yml)
for Detekt which is used when we're gathering code quality metrics for the code quality dashboard. The reason for this
is to have a setup that isn't as restrictive as our main config file.

### Quality of Life
There's an IntelliJ Idea plugin available for Detekt that we highly recommend you to use. You can install it by going
to `Android Studio > Preferences > Plugins` and search for Detekt.  
#### Configuration
After installing the plugin and restarting your IDE, and navigate to `Android Studio > Preferences > Tools > Detekt` 
and from here make sure that you're using this configuration:
```text
✅️ Enable Detekt
🔲 Treat Detekt findings as errors
🔲 Build rules upon the default configuration
✅️ Enable formatting (ktlint) rules
🔲 Enable all rules

Configuration File: /${PROJECT_PATH}/config/quality/detekt/detekt.yml
```

#### Auto Format
There's an auto formatter tool available for Detekt which you can use to automatically fix some of the formatting
issues, like unused imports. To do this run this command:
```shell
./gradlew modulename:detektDebug -PautoFormat=true
```
Simply replace `modulename` with the name of the module that you want to perform auto formatting in. For example:
```shell
./gradlew analytics:detektDebug -PautoFormat=true
```
*Tip: In case you're running this command from the terminal interface in your IDE, you can press `⌘ + ↵` instead of just `↵`
to run the Gradle task through IDE.*

### Ignore
Let's say Detekt is giving you a warning because of `LargeClass`, and you want to ignore this warning. Currently you
can use `@Suppress("LargeClass")` annotation on the class that you want to be ignored for this rule. Note that this
approach isn't recommended and we highly suggest to document these as tech debts and fix them in the future.
#### Baseline
Each module has a `detekt-baseline-debug.xml` file which are the legacy warnings that are ignored. When we introduced
Detekt into the project obviously we had dozens of warnings, and we ignored them through these baseline files so we
could focus on not introducing new warnings, and in the meantime have some sort of documentation of the currently
existing warnings that each module has. ***These baseline files shouldn't be edited manually and by hand.***
