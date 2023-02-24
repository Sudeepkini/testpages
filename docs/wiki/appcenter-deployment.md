# AppCenter

App distribution tool. Testers can download and install the app on their device.

Go to https://appcenter.ms/apps and login with SSO (work account)

# Bitrise

* It is a CI/CD platform focused on mobile development.
* It runs UI tests (smoke tests) on every PR and every commit to development.
* Visit https://app.bitrise.io/dashboard/ login using SSO with organization name `Delivery Hero SE`.

You can use it to:

* Update translations (get the latest values from webtranslateit)
* Upload staging builds to hockey (It will also comment on the JIRA ticket with the build links (if there’s a ticket code in the branch name))

**Step by Step running a workflow:**

![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/wiki-images/bitrise_1.gif)

**Customizing AppCenter build for specific brands:**

By default, running the `Upload-All-Production-on-HockeyApp` would build all brands proguarded and debuggable. If you want to build one or more specific brands, you can use the "Advanced" settings on Bitrise when triggering the build using the "Start/Schedule a Build" button as shown below:

Possible params:

`b` -> Build only specific brands(comma separated). If not specified it will build all brands

`debug` -> It's `false` by default for release builds. You can enable it with `debug:true`

`proxyable` -> It's `true` by default for release builds. You can enable it with `proxyable:false`

`proguard` -> If you want to debug release builds you will need `proguard:false` so you can debug the code. Default is `true` for release

`disableMasCheck` -> If you want to disable the MAS checks for release builds, you will need `disableMasCheck:true`. Default is `false` for release, which means MAS is enabled. See [Link](https://urban-journey-e7eda8fe.pages.github.io/modules/app-security/) for context.

![bitrise-brand-specific](https://user-images.githubusercontent.com/8232573/94162843-71b5c780-fe87-11ea-96a7-9fb293f0eb6f.gif)

Don't forget to press the "Add Environment Variable" button!

Key: `PARAMS`
Value: `b:foodpanda debug:false proguard:false`

For triggering the build for more than 1 brand, just append the new brand names in a comma-separated list:

<img src="https://user-images.githubusercontent.com/8232573/94162839-70849a80-fe87-11ea-9aa1-dd056ba92ced.gif" width=300 height=350/>

Key: `PARAMS`
Value: `b:foodora,damejidlo debug:false proguard:false`

More information about the script and possible parameters can be found [here](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/fastlane/fastlane_upload_hockey.rb#L192).

By default, if the key `PARAMS` isn't added, this workflow would trigger builds for all brands, debuggable, mas enabled and proguarded.

NOTE: Also this customization apply for `Uploading-All-Staging-To-HockeyApp`
