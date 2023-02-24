# Welcome to B2C Wiki!

B2C stands for "Business To Client" in case you were wondering!

This wiki is not a complete guide from A to Z Yet, it has most of the necessary information you might need to start working on this project.
Improving this wiki needs some effort from everyone. So please feel free to take some notes while reading through it and share them with us to make this guide as useful as possible.

Mobile Chapter shared [google drive](https://drive.google.com/drive/u/0/folders/1L2zopKzEyFWFo6F-YPOZ4E8vgEcv_VcO) (If you don't have access please reach out to Mobile Infra team) contains useful links and [folder](https://drive.google.com/drive/u/0/folders/13Zgn7KJe3dbHywShF7OjlP8CIObnxy_C) with recordings from Android chapter meeting presentations. Onboarding App Flow [video](https://drive.google.com/file/d/1hCOPwSlNGSjKymvl7V0lEoxZVhm4CVwB/view?usp=sharing) is usefull for new joiners to get the first impression of how the mobile apps work.

## 1 - Let's get started!
To get started, I would recommend taking a look at how to set up your working environment. In case you encountered any problem during this process, please feel free to contact any of your teammates to help you on fixing it.

[Setup Environment](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/setting-up-environment/)

Also if you're having difficulty to search the class name of the activity or fragment that you want to modify, you could simply go to the page that you're looking for in the apps and open the notification bar, it will show foodpanda notification that contains the class name of the opened screen like this:


<img width="372" alt="Screenshot 2022-01-13 at 10 02 56 AM" src="https://user-images.githubusercontent.com/78725129/149253047-a8bdcb9d-3188-4341-9634-9ddb281a4e28.png">

## 2 - Dependencies And Configurations
Now let's take a look at how our project is structured before moving on to any other topic. What are the dependencies and the configurations needed for the project to be runnable? All this information and more can be found here.<br/><br/>
[Pandora Android Repositories](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/pandora-android-repo/)<br/>
[App Configurations](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/app-configuration)

Note that in the organization we have our own [Design System](https://github.com/deliveryhero/pd-pretty/tree/master/core/src/main/java/com/deliveryhero/pretty/core) that we implemented in our project and it's called "Bento" so that the design team can trigger app-level UI changes by updating these components without having the squad-devs to make changes every time. This also allows us to have a consistent UI across the app for all brands.
In our App we refer to Bento components with "Core components"

## 2 - Architecture
Next, you would be reading more about the architecture we use in the app, We don't force you strictly to favor one architecture over the other, but, It's very important to understand each existing architecture and see the potential of it and in which case we are encouraged to use it. Try to understand what is the problem each architecture is trying to solve and apply it to the project based on your needs. Currently, we are using the Clean Architecture defined by Uncle Bob in two forms (MVP, MVVM). So far, we haven't face any problem using this architecture. [Read more about it here](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/architecture/)

## 3 - Localization
How does our project support many different languages? We have our own solution for that. You can learn more about it [here](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/application-localization)

## 4 - Code Readability and Style
>“The ratio of time spent reading (code) versus writing is well over 10 to 1 … <br/> (therefore) making it easy to read makes it easier to write.” <br/>
>— Robert C. Martin

It's very important to pay attention to your coding styling and the naming convention. We don't want each developer to implement their own style which will lead to a much harder code to read due to the differences. It's recommended that we use the same approach.
You can read more about this topic [Here](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/code-readability-style/). If you have other opinions about this topic I would strongly suggest that you share them with the team so we can all be alligned<br/>

Other Useful links<br/><br/>
[Kotlin Offical Guidelines](https://kotlinlang.org/docs/reference/coding-conventions.html), <br/>
[Google - Core App Quality](https://developer.android.com/docs/quality-guidelines/core-app-quality),<br/>
[Design Quality Guidelines](https://developer.android.com/design/index.html)<br/><br/>
Also, if you pay attention to the code style when doing code review you might learn new things from your peers!<br/>

Some books to refer
1. Effective Java - Joshua Bloch
2. Refactoring: Improving design of existing code -  Martin Fowler
3. Psychology of Code Readability - Egon Elbre

## 5 - Static Analysis
Static analysis will help us to follow a specific set of rules that we define which saves us plenty of time when doing code review and help us to maintain high standard code quality. To read more about this topic I would suggest that you read this [guide](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/static-analysis/).

## 6 - Development Tools
Every team in our company is using a different set of tools that fit their needs. We are trying to adapt the best tools available out there to help us be on the top. Sometimes we develop our own set of tools to eliminate the dependency on third parties. You can learn more about these tools in this [section](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/development-tools/)

## 7 - Testing
Every engineer knows why testing is very important for the product! That's why we have invested some time and effort in developing our own End to End testing framework. You will learn more about this framework and how to use it in this [Workshop](https://github.com/deliveryhero/pd-android-ui-testing-workshop). <br/>
Please take a look at these links too <br/><br/>
[Application Tests](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/app-testing/)<br/>
[TA Performance Checks](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/ta/)

## 8 - Let's contribute!
After reading through all the previous topics, now it's time to contribute to our awesome project! But before sending your first Pull Request, you need to go through this [part](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/code-review-pull-requests)

## 9 - App Release Process
Maybe you have sent your first PR by now and you're wondering what is the next step and when your code changes will be released! To answer this question, first, you need to understand how our release process works. You can learn everything about it in this [section](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/app-release-management)

## 10 - Brand migration
This topic is not a Must to read especially if you are a new joiner. But, if you want to learn more about how we migrate a new brand into our platform [Pandora], then this will be very helpful to read [Brand migration](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/brand-migration)
https://deliveryhero.github.io/pd-mob-b2c-android/wiki/home/
## 11 - Miscellaneous
Here you can find some interesting topics that I highly recommend to read. You will find some tips and recommendations from our team, things to do more, things to avoid, and many other useful topics that you shouldn't miss!<br/><br/>
[Coroutines](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/coroutines-guide/)<br/>
[Unified Error Handling API](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/unified-error-handling/)<br/>
[Helper modules in metalcore](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/metal-core-helper-modules)<br/>
[Deeplinks](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/deeplinking)<br/>
[Miscellaneous](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/miscellaneous/)<br/>
[Using Static Images as Resources](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/using-static-images/)<br/>
[Analytics Tracking](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/analytics/)<br/>
[Logging](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/logging/)<br/>

## 12 - Suggestions!
If you see any missing parts in our guide or maybe you think we need to improve on a topic, please add your suggestions here, and let the team know about it. We value any feedback or initiative 💪<br/>
[Open Topics List and Improvement suggestions](https://deliveryhero.github.io/pd-mob-b2c-android/wiki/open-topics-improvements/)
