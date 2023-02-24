## Introduction
You can find more information for deeplinks types and links structure on this [Confluenge Page](https://confluence.deliveryhero.com/pages/viewpage.action?spaceKey=GLOBAL&title=Product+Marketing+-+Deeplinks).

We use [android-deeplink](https://github.com/hellofresh/android-deeplink) library as the basis for our deeplink framework which offers a simple and flexible API

When setting up a new deeplink in the app. There are a few things to keep in mind
- All route implementations should extend from `DhRoute`
- Routes must be registered to the DeepLinkParser in DeepLinkModule
- All Routes return an instance of `DeepLinkSpec` which we can use to describe the requirements for such route. For example, what intent should be launched, whether this destination requires user to be logged in, etc

## Testing a new deeplink

#### Using ADB command
You can quickly execute this ADB command to launch the app once your new deeplink setup is complete
```
adb shell am start -a android.intent.action.VIEW -d "<URI>"
```
Where URI could be something like `foodpanda-st://?c=sg\&s=h` for example. Note the **\\** before **&**

#### Using Android Studio
This could also be configured directly from android studio by modifying/create a new Run Configuration

<img src="https://github.com/deliveryhero/pd-mob-b2c-android/raw/development/wiki-images/studio-deeplink.png" />

## Dynamic links
We support dynamic links using Firebase/Adjust/Branch. While Branch are being slowly phased out, Firebase remains the preferred option, and Adjust links will already be unwrapped before they trigger the app, so we don't need to do anything

When the app is launched via a dynamic link, we will automatically unwrap such link to a proper deeplink. This means Route implementations only need to support deeplinks as usual and never have to worry about dynamic links.

## Internal navigation via deeplink
There are features that may wish to take advantage of deeplink as an approach for _dynamic_ navigation within the app. We have [this extension](https://github.com/deliveryhero/pd-metalcore-android/blob/master/commons/src/main/java/com/deliveryhero/commons/Deeplink.kt) specifically designed for such purposes. It will handle automatic unwrapping (if needed), and eventual navigation to the expected destination.

An advanced use-case may prefer to skip launcher activity altogether for a relatively smoother navigation experience. In such situation, `DeepLinkProcessor` may be injected directly and the result must be handled manually. Also, any dynamic link will have to be manually unwrapped before passing it to DeepLinkProcessor

**Note**: It is highly recommended to use the app's deeplink scheme for internal navigation to avoid extra unwrapping overhead
