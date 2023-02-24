# Charles

Charles is an HTTP proxy / HTTP monitor / Reverse Proxy that enables a developer to view all of the HTTP and SSL / HTTPS traffic between their machine and the Internet. This includes requests, responses and the HTTP headers (which contain the cookies and caching information)

[More info:](https://www.charlesproxy.com/)

To debug a release/production build, make the following 3 changes in your code + the 4th one to see the API requests on Charles:

**1. build.gradle (app)**

![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/wiki-images/charles_1.png)

**2. ApplicationModule.java**

![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/wiki-images/charles_2.png)

**3. ActivityLifecycleTracker.java** (Optional - Only needed to enable the blue RI tracking widget)

![](https://github.com/deliveryhero/pd-mob-b2c-android/blob/development/wiki-images/charles_3.png)

**4. To be able to SSL proxy requests on charles**

Replace `network_security_config.xml` with:

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config>
        <trust-anchors>
            <certificates src="system" />
            <certificates src="user" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

# Flipper

Flipper is a platform for debugging mobile apps on iOS and Android. Visualize, inspect, and control your apps from a simple desktop interface. You can see the layout hierarchy, memory leaks, network calls, shared preferences. https://fbflipper.com/

# Firebase

Powerful tool that gives you functionality like analytics, databases, messaging and crash reporting so you can move quickly and focus on your users.

What we use from firebase:

* Crashlytics
* Performance Monitoring
* Test Lab
* Google Analytics
* Remote Config

**Active Firebase Projects per brands:**

* Foodpanda APAC, Foodora NO: [BFD - Foodora](https://console.firebase.google.com/u/0/project/android-foodora)
* Foodora SE (Previous OnlinePizza): [BOP - OnlinePizza](https://console.firebase.google.com/u/0/project/online-pizza-apps)
* Foodora FI (Previous PizzaOnline): [Pizza-Online](https://console.firebase.google.com/u/0/project/pizza-online-ea3be)
* Foodora DK (Previous Hungry): [HungryGroup](https://console.firebase.google.com/u/0/project/hungrydk-ab4c9)
* Foodpanda EU (Previous Netpincer): [netpincer](https://console.firebase.google.com/u/0/project/netpincer-1239)
* Damejidlo: [DHH - Damejidlo - Firebase](https://console.firebase.google.com/u/0/project/dj-client-stage)
* Mjam: [DHH - Mjam - Firebase](https://console.firebase.google.com/u/0/project/mjam-2a6af)
* Yemeksepeti: [Yemeksepeti](https://console.firebase.google.com/u/0/project/yemeksepeti-pandora)
