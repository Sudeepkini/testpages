### Static analysis in Pandora

We're using a bunch of different tools to run analysis in our Project

* Detekt
    * Reports common Kotlin issues and code smells
* KtLint (as a part of Detekt)
    * Mostly code style, extra spaces and new lines
* Checkstyle
    * Same as KtLint but for Java
* Android Lint
    * Reports common issues for Android, works in both Java and Kotlin
* Jacoco Report
    * Reports unit test coverage of modules, works for both Java and Kotlin files

These analysis run parallel to unit and UI tests, so it won't make PRs stay open for longer

### Enforcing rules before opening PR

In order to catch these common code style issues you can install the `pre-commit` hook to your local repository. It will run a code style check only on VCS changed files and report any errors on `html` format.
If the tool reports some issues which you feel don't need to be fixed or are existing issues, use `git commit --no-verify` to commit.

#### How to install the pre-commit hook

* Go to `${projectRootDir}/config/gitHooks` and copy the `pre-commit` file
* Go to `${projectRootDir}/.git/hooks` and paste the file you copied
* Make the `pre-commit` file executable (in mac: `chmod +x pre-commit`)
* You're good to go

### Custom DH code style rules

* Don't use `printStackTrace()`
    * Reports usages of `printStackTrace()` and advises to use `Timber.e()` instead, it also has a quick fix option in Android Studio
* New line before return
    * Reports the common issue we have with new lines before return statements
* Warn if we're not handling errors in Rx streams
* Use `@SerializedName` for `ApiModel`
    * We should always use `@SerializedName` for api models, because ProGuard obfuscates these classes and we end up with hard to catch bugs
* Use pretty widgets instead of android widgets
    * We have problems some time when we don't use widgets like `DhTextView` because of translations
* Check if `android:src` is used instead of `app:srcCompat` and `android:drawableStart` is used instead of `app:drawableStartCompat` as a warning or if we can detect if the drawable is vector then could be reported as error.
* Empty line between class name & 1st method signature
* Empty line between methods signatures especially in interfaces
* Alert when we add class with the legacy  package de.foodora.android

#### Running custom rules locally

In order to run these custom rules with the rest of Android Lint in Android studio, you have to download **Android SDK Command-line Tools (latest)**
![Android tools latest](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/wiki-images/android-tools-latest.png)
once you have this, all you have to do is build the project

If you want to run static analysis from the command line then you can use the following commands:
```
// To run for the whole project
./gradlew lintDebug lintFoodpandaDebug

// To run for your module
./gradlew {your_module}:lintDebug

// For app module
./gradlew app:lintFoodpandaDebug
```

#### To be implemented

* Validate xml view ids
    * We have a rule for setting xml view ids
* Warning of memory leakage on using anonymous classes (object expressions) inside android framework class e.g Activity/Fragment
* Add custom rule to check is multi line can fit into single line with 120 chars limit
* Consider [Cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) metric which detects high level of code nesting
* Consider spelling issues detection
* Rename modules with prefix of `feature` or `library` and check in case any feature module is added as dependency to another feature module. Maybe as another idea separate feature lib directory can be better.
* Make teamplate module and repository as well.
* Check for new permissions introduced by each PR using adb command. Keep list of previous existing permissions and compare for new ones.
* Detect if tests in TA have new `@SmokeTest` new annotation which will be extending the smoke test suits and increase build time. In this case we will need to mention TA team on the PR and ask them to take an extra look and approve.

***

Please add more ideas for rules in the section above
