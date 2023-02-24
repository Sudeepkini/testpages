# [Mob -b2c - Android](https://github.com/deliveryhero/pd-mob-b2c-android)

This repository contains the main application module which is the core B2C android app together with all feature and library modules.

The main project follows a multi-module architecture with multiple independent feature modules and shareable library modules.
**Every module must have it's own Readme file** containing: Owner/s of the feature (names and/or teams), responsibilities and description.
Module naming convention:
* Feature modules - `feature-[module name]`
* Library modules - `library-[modules name]`

The repository has dependencies from external modules and other repositories

* Metalcore
* Pretty
* XPay
* Perseus Android SDK
* Static S3 Config

**Note**: The module bootstrap script must be used when adding a new module in b2c. For more detail on how to use this script, open a terminal from b2c project root, and run
```sh
./scripts/create-module.sh --help
```

## Dependency Graph for Internal Modules (Automatically updated daily)

### Legend
![Legend](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/gradle/dependency-graph/legend.dot.png)

### Dependency Graph with Owner modules grouping
[![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/gradle/dependency-graph/projectSquadCluster.dot.png)](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/gradle/dependency-graph/projectSquadCluster.dot.png)

### Dependency Graph
[![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/gradle/dependency-graph/project.dot.png)](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/gradle/dependency-graph/project.dot.png)

### How to generate these graphs.
In order to generate these graphs you can run `./gradlew projectDependencyGraph`.

Note: the command uses [`graphviz`](http://www.graphviz.org/) to install on mac you can use `brew install graphviz`

This task has different optional params:
- `includeApp` (it will show the `app` module)
- `includeSquadClusters` (it will show aggregations of the modules per squad)
- `owners` - provides a filter for displaying only modules related to the given owners.
- `modules` - provides a filter for displaying only the modules given.

### Filtering by owner
In case you want to generate this graph to see modules related to your squad, you can add a filter to the graph generation task. For example, if you are part of the `Search` squad (as in the `moduleSquadOwner property in the `gradle.properties`), you should run:

```
./gradlew projectDependencyGraph -Powners="Search"
```

## [Metal core](https://github.com/deliveryhero/pd-metalcore-android)

This is a separate repository that contains different modules that are sharing the same code logic to be exposed to any external dependency. For example, it has the `entities` module which contains api models to parse API responses.

Below you can find some of the shared modules:

* `commons`: it has some logic that is forced to be used in application module and any external module / repository
* `localization`: which has implementation of providing dynamic translations to the app
* `encryption`
* And more...

## [Pretty](https://github.com/deliveryhero/pd-pretty)

This is the design system repository that contains our shared UI widgets that could be used in any module. Customizing any widget there will reflect on all external dependencies and it is a good central mechanism to enforce our design system.

Our design system has predefined typography, icons, colors, and dimensions. This way, every widget in the app has to extend from our UI component and it must define the correct typography to use.

If you have any new widgets or modifications it has to be aligned with the design team and to be approved from them and the tech team as well.

Adding third parties to pretty is not recommended as it increases the application build time and the final app size.

[Pretty wiki](https://github.com/deliveryhero/pd-pretty/wiki)

## [XPay - Wallet & payment selection](https://github.com/deliveryhero/pd-mob-xpay-android)

Mainly is a separate repository for the Wallet project as it is a major feature that affects the whole app and we have different teams to collaborate on we decided to keep it in a separate repository. Deploying new versions from wallet is critical and it has to be approved from everyone.

## [Perseus](https://github.com/deliveryhero/pd-perseus-android)

Internal analytics SDK for Android that is connecting with GTM through Function Call feature and re-receives events from Google Analytics that are dispatched from the application and later dispatched through API call to Perseus services.

## [Static S3 config](https://github.com/foodora/static.fd-api.com)

This is a repository that provides a configuration per country and client. For example, the `Pickup` feature can be enabled in specific countries and only for specific clients such as Android, iOS or Web.

---

## Creating Releases for these repositories

These repositories require manual deployment and release tags: every external repository requires release tags except S3 config as it is consumed through API call.

### How to create release from this repositories?

There are two ways of creating releases first one is local and the second way is to build and upload release on Github Packages.

**How to build local version?**

* Select your desired repository
* Make sure to pull from the main branch to have the recent updates
* Create a branch for your changes
* Go to `{PROJECT_ROOT}/gradle.properties`
* Please use semantic versioning when increasing the version.
* Go to your project gradle tasks / tasks / publishing and execute -> `publishToMavenLocal`
* Switch to application project and build the project with the new local build number
* Test your features against the new local version

**Creating Releases on Github Packages**

If your pull requests and tests against the local builds are approved be aware if merging your third parties pull requests to the master branches as it will break the application if anyone accidentally created a release with your code but rather make sure to get your application changes approved as well and merge your pull requests once and ask your colleagues to rebase.

**Another useful trick** to build remote release from your open PR without merging it to master, get the first 7 characters of your branch commit HEAD and put it as a release tag with title example `aaabbbx-SNAPSHOT` and this will trigger a new build on Github Packages that you can use as a version in your open PR in the main repo where this version needs to be tested.

**One more tip**: Use testing by commit id to make sure that your application will work after your merge the dependency pull request and to make sure that unit and ui tests passes as well.

**Creating new release**
* Go to your repository in Github
* Switch to releases
* Click draft release
* Add your release number, title and notes if any
* Publish release
* Add the new release number to your project and merge it to your main branch

**How to add the repository in your build gradle?**

Go to your module / app `build.gradle` file and add the following:

```
implementation "com.github.deliveryhero.pd-metalcore-android:pandora-api:{Your_Version}"
implementation "com.github.deliveryhero.pd-mob-xpay-android:wallet:{Your_Version}"
implementation "com.github.deliveryhero.pretty:ui:{Your_Version}"
```

Main idea is to add your repository link and follow it with your target module and build number

**Important!!!**: It is always better to deprecate the old logic and introduce the new one instead of modifying  what's already used by the clients as it is considered a breaking changes so always favor deprecation over modification.

## **Follow [semantic versioning](https://semver.org/) for libraries and SDKs**

Version numbers take the form `X.Y.Z` where:

**Major version (X)**: Must be updated if changes are not backwards compatible.

**Minor version (Y)**: Must be updated if changes include backwards compatible new functionality in the public API, newly deprecated APIs, or substantial new functionality/improvements to private code.

**Patch version (Z)**: Must be updated if changes include backwards compatible bug fixes. **Important:** As previously was agreed to increment patch only for versions that are fixing bugs for release related topics. Only increment if this version will be merged to a release branch.
