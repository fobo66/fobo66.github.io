---
title: "Migrate from Play Services Location to Android's LocationManager API"
published: false
---

Hello! After recent announcement from Microsoft about Android apps support in Windows 11, I was excited to try to publish my app to Amazon Appstore, so I can discover more form factors and attract more users. However, there was one caveat: Windows can only run apps without Google Play Services dependency, since it will not be present in Windows and in Amazon Appstore. So, I started looking for workarounds.

My app was using only `FusedLocationProviderClient` API from Play Services, and only to load current location of the device, so I found a way how I can achieve similar results in more Kotlin-friendly way with help of existing Android SDK LocationManager API. It turns out that there was not so much info about Location API in Android, since even [official docs](https://developer.android.com/guide/topics/location/migration) are advising to use Play Services.

In this article, I will tell a bit about Play Services and `LocationManager`, warn you about some caveats of Android's Location API and provide the code that loads current location with `LocationManager`.

## What's wrong with Play Services?

Play Services is a separate app that allows devs to use various aspects of Android OS independently of vendor modifications. It contains various useful services besides location. Quoted from its [Play Store page](https://play.google.com/store/apps/details?id=com.google.android.gms&hl=en_US&gl=US):

> Google Play services is used to update Google apps and apps from Google Play. This component provides core functionality like authentication to your Google services, synchronized contacts, access to all the latest user privacy settings, and higher quality, lower-powered location based services. Google Play services also enhances your app experience. It speeds up offline searches, provides more immersive maps, and improves gaming experiences. Apps may not work if you uninstall Google Play services.

Looks promising, but there are concerns about it in the industry. First, Play Services are installed on the most of the Android devices, but not on all of them. For example, some Mainland China vendors don't have Play Services preinstalled on their devices, since Google is banned there, and users mostly can live without them, since there are analogs in China. And second is that it's still a separate app, but it is treated as a part of Android OS. It means that developers rely on them heavily, and if there's no Play Services on some device, apps will break and won't be able to work. And there are almost no workarounds for that. So it looks like severe vendor locking, which is never a good thing. It's not exactly a monopoly, because there are some alternatives, but still.

From developer's perspective, I can provide my own concerns, since I cannot speak for everyone. First thing that bothers me as a dev is Play Services' Task API, that is quite inconvenient to work with in today's codebase. It doesn't fit well with coroutines or RxJava. Plus it's nullable API, so you can get null result from some request, and e.g. in RxJava it's not acceptable answer, despite being valid from Play Services perspective. Because of that devs are forced to use some workaround, sometimes dirty ones, which is not a sign of a good API, in my opinion. I understand that it was created before RxJava and coroutines took over Android development, and at that time it was decent solution, compared to AsyncTasks, but nowadays it feels quite outdated and not very suitable.

Second thing is the license of Play Services libraries. Google seems to use some kind of open license with limitations, similar to AOSP. You can find source code of Play Services libraries in the AOSP repo, but it contains some proprietary code anyway, and it's obfuscated on the client side. That's more of a nitpick, since you can use these libs freely in any application without limitations, but for critical part of the infrastructure of many Android apps, I feel like it should be more open, so community can at least see how it all works, or fix bugs without relying on Google to do so.

Third thing is that Play Services APK is immense. It takes up too much space on the device and contains a lot of things that no app might use on the device. I understand that it's done to reduce sizes of other apps by moving common code to the one app, but I would prefer to find better way here. For lower end devices, Play Services can easily take over the whole available space, so user will struggle installing anything else. Situation with this worsens every year.

## What is LocationManager?

`LocationManager` is a system service on Android available since API 1. It's responsible for various operations related to device location. In the beginning of Android, people were using it for everything. However, it was requiring too much boilerplate code even for simple operations. You can still find some old Android docs for it on MIT website to see how complicated was the setup of it. Boilerplate code is not fun, of course
