---
layout: post
title: React Native - Onesignal Proxy
date: 2019-07-30 10:43:47 +0300
access: public
comments: true
categories: [react-native]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

> <https://github.com/geektimecoil/react-native-onesignal/issues/615#issuecomment-416441323>
>
> You would need to remove OneSignal from your build.gradle and you would need
> to manually download the Android and iOS native SDK’s (links below) and modify
> them to change the URL.
>
> Once you change the URL, you’ll need to recompile both of them, compile
> Android SDK into a jar, and you can compile the iOS SDK into a framework or a
> static library, or even host it as a private cocoapod.

## Android

### OneSignal-Android-SDK

- clone <https://github.com/OneSignal/OneSignal-Android-SDK>

  NOTE: there's no need to fork this repo - we're not going to commit and push
  changes.

  ```sh
  $ git clone git@github.com:OneSignal/OneSignal-Android-SDK.git
  $ cd OneSignal-Android-SDK
  ```

- change `BASE_URL` in `OneSignalRestClient` class

  ```diff
    // onesignal/src/main/java/com/onesignal/OneSignalRestClient.java

  - private static final String BASE_URL = "https://onesignal.com/api/v1/";
  + private static final String BASE_URL = "https://<PROXY_HOST>/onesignal/api/v1/";
  ```

- create AAR file

  > <https://developer.android.com/studio/projects/android-library>
  >
  > ```groovy
  > apply plugin: 'com.android.library'
  > ```
  >
  > The entire structure of the module remains the same, but it now operates as
  > an Android library and the build will now create an AAR file instead of an
  > APK.

  ```sh
  $ cd OneSignalSDK
  $ ./gradlew build -x test
  ```

  output files are located in _OneSignalSDK/onesignal/build/outputs/aar/_ - we
  need _onesignal-release.aar_ file only.

### react-native-onesignal

- fork and clone <https://github.com/geektimecoil/react-native-onesignal>

  ```sh
  $ git clone git@github.com:<MY_COMPANY>/react-native-onesignal.git
  $ cd react-native-onesignal
  ```

- copy Android SDK JAR file to _android/libs/_

  > <https://stackoverflow.com/a/21485222/3632318>
  >
  > The AAR file consists of a JAR file and some resource files

  - extract _classes.jar_ from _onesignal-release.aar_ file
  - rename _classes.jar_ to _onesignal-<MY_APP>.jar_ (or the like)
  - copy _onesignal-<MY_APP>.jar_ to _android/libs/_

- replace original Android SDK dependency in _android/build.gradle_

  1. <https://developer.android.com/studio/build/dependencies>
  2. <https://life.nimbco.com/2013/09/referencing-local-aar-files-with-android-studios-new-gradle-based-build-system/>

  NOTE: make sure to remove `exclude group` line - OneSignal classes are not
  found otherwise.

  ```diff
    // android/build.gradle

  - implementation('com.onesignal:OneSignal:3.11.1') {
  -     // Exclude com.android.support(Android Support library) as the version range starts at 26.0.0
  -     //    This is due to compileSdkVersion defaulting to 23 which cant' be lower than the support library version
  -     //    And the fact that the default root project is missing the Google Maven repo required to pull down 26.0.0+
  -     exclude group: 'com.android.support'
  -     // Keeping com.google.android.gms(Google Play services library) as this version range starts at 10.2.1
  - }
  + implementation files('libs/onesignal-<MY_APP>.jar')
  ```

  or else you might try to reference AAR file directly - see the linked post #2.

- commit and push changes

### your application

- use fork of `react-native-onesignal` npm package in _package.json_

  ```diff
    // package.json

  - "react-native-onesignal": "^3.2.14",
  + "react-native-onesignal": "github:<MY_COMPANY>/react-native-onesignal",
  ```

  ```sh
  $ yarn
  ```

- build and run Android application in emulator

  ```sh
  $ react-native run-android
  ```
