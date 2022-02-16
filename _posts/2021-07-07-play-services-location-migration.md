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

Looks promising, but main concern about Play Services in the industry is that it is treated as a part of Android OS. It means that developers rely on them heavily, and if there's no Play Services on some device, apps will break and won't be able to work. And there are almost no workarounds for that. So it looks like severe vendor locking, which can be dangerous in the future. It's not exactly a monopoly, because there are some alternatives, but still, there are cases when Play Services are not installed on the device or there is no way to install them. Examples are Amazon devices that don't have Play Services or Huawei devices that cannot have Play Services at all due to sanctions.

From developer's perspective, I can provide my own concerns, since I cannot speak for everyone.

1. _Play Services' Task API_ that is quite inconvenient to work with in today's codebase. It doesn't fit well with coroutines or RxJava. Plus it's nullable API, so you can get null result from some request, and e.g. in RxJava it's not acceptable answer, despite being valid in some cases (Location that you retrieve can be null). Because of that devs are forced to use some workarounds, sometimes dirty ones, which is not a sign of a good API in the first place, in my opinion. I understand that it was created before RxJava and coroutines took over Android development, and at that time it was decent solution, compared to `AsyncTask` and bare threads or Executors, but nowadays it feels quite outdated and not very suitable for modern apps' approaches to async work, although you can use some adapters for it for Rx or coroutines.

2. _License of Play Services libraries._ Google seems to use some kind of open license with limitations, similar to AOSP, for its SDKs. You can find source code of Play Services libraries in the AOSP repo, but it contains some proprietary code anyway, and it's obfuscated on the client side. Also, the Play Services app itself if a closed-source, so we can only guess what's going on there. That's more of a nitpick, since you can use these libs freely in any application without limitations, but for critical part of the infrastructure of many Android apps, I feel like it can be more open, so community can at least see how it all works, or be able to send patches without relying on Google to do so.

3. _Play Services APK is immense._ It takes up too much space on the device and contains a lot of things that no app might use on the device. I understand that it's done to reduce sizes of other apps by moving common code to the one app, but I would prefer to find better way here. For lower end devices, Play Services can easily take over the whole available space, so user will struggle installing anything else. Situation with this worsens every year, e.g. recently they decided to include TensorFlow Lite runtime in the Play Services to reduce the size of the apps that use it, so they will only need to store models.

Play Services client-side SDKs are usually working in the same way: they create some intents to Play Services app and load data from there. It is well hidden behind obfuscation and tedious interface of aforementioned Task API. They are quite thin because of that, but still.

So, if it all is concerning you as well, let's check what options we have:

* Huawei Mobile Services (HMS) – great set of open source tools developed by Huawei to replace Play Services. People tend to use them both in one app, e.g. by having special flavor that uses HMS and other flavor that uses Play Services. But it is available mostly for Huawei devices, so we cannot use it everywhere. Well, we can, probably, because HMS SDK can download and install HMS APK to use it, but it can have compatibility issues with non-Huawei devices. Also it is quite similar to Play Services, so drawbacks will also be the same.
* MicroG – open source replacement for the Play Services. It's a great tool that masks as Play Services and does the same job mostly, but it requires root access to be properly installed, and thus it's mostly available in custom ROMs used by enthusiasts.
* Write custom solution – best option in terms of control and matching requirements, but it can take a lot of time depending of what part of Play Services you would like to replace and what resources you have at your disposal, e.g. custom backend for the payments processing or some maps provider that load highly detailed maps from your own satellite (just kidding).

Turns out that we have only limited possibilities to replace Play Services, or at least some parts of it. Let's turn back to the initial point of replacing Play Services Location API.

## What's this with the location?

Location handling in Android is historically not so well-designed. [`LocationManager`](https://developer.android.com/reference/android/location/LocationManager) is a system service on Android available since API 1, and it was used for various operations related to device location since the beginning. In the beginning of Android, people were using it for everything. However, it was requiring too much boilerplate code even for simple operations. You can still find some [old Android docs](https://stuff.mit.edu/afs/sipb/project/android/docs/training/basics/location/locationmanager.html) for it to see how complicated was the setup of it. Boilerplate code is not fun, of course, but in this case it's the type of code you would write once and forget about it for a long time. Google [suggests](https://developer.android.com/guide/topics/location/migration) not to use it directly anymore because they created a replacement. But since we decided not to use this replacement, let's find out more about how `LocationManager` works.

`LocationManager` operates on a very low level of location data. There are various providers of the location info, e.g GPS, cellular networks, Wi-Fi, etc. And `LocationManager` does not automagically gathers info from all of them, we need to explicitly ask various providers of the location data for the location, and these providers may have very old location, or no location at all. Also, `LocationManager` may not be available on the device (although with little probability) or location access can be disabled on device, or some provider can be disabled for some reason (and won't give us any data because of that), so we need to check for these cases as well. In short, there are tons of nuances that we need to account for.

## How to request current location with LocationManager

Simplest way to get location would be to query each of the available providers for the last location they've got. It will be something of a simple traversing of the list of available locations. If we require the most recent location, we can request current location from each of the available providers, but it's more asynchronous operation, so it can take some time. We can just try to request some particular provider that is reasonably precise for us, e.g. GPS or cellular, but in my experience they are not so reliably available on device, sometimes GPS can be disabled with no reason or cellular will take some time to determine location.

Here is the code I ended up using for getting current location:

```kotlin
private val noLocation by lazy {
    Location(0.0, 0.0)
  }

@RequiresPermission(Manifest.permission.ACCESS_COARSE_LOCATION)
suspend fun resolveLocation(): Location {
    val locationManager = context.getSystemService<LocationManager>() ?: return noLocation

    return if (LocationManagerCompat.isLocationEnabled(locationManager)) {
        var location: android.location.Location? = null

        withContext(ioDispatcher) {
            location = locationManager.getProviders(true)
                .asSequence()
                .map { locationManager.getLastKnownLocation(it) }
                .filterNotNull()
                .find {
                    SystemClock.elapsedRealtimeNanos() - it.elapsedRealtimeNanos <= locationFixTimeMaximum
                }
        }

        location?.let {
            Location(it.latitude, it.longitude)
        } ?: noLocation
    } else {
        noLocation
    }
}
```
