---
layout: post
title: React Native - OneSignal Proxy
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

## Nginx

1. <https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/>
2. <https://vc.ru/claim/76946-servis-push-uvedomleniy-onesignal-nedostupen-v-rossii?comment=1334456>

configure Nginx as a reverse proxy to proxy requests to `https://onesignal.com`:

```diff
  # /etc/nginx/sites-available/<MY_APP>_production

  server {
    listen 443 ssl;

    # ...

+   location /onesignal/ {
+     proxy_pass https://onesignal.com/;
+   }
  }
```

## OneSignal-Android-SDK

- fork and clone <https://github.com/OneSignal/OneSignal-Android-SDK>

  ```sh
  $ git clone git@github.com:<MY_COMPANY>/OneSignal-Android-SDK.git
  $ cd OneSignal-Android-SDK
  $ git checkout 3.11.1
  $ git checkout -b <MY_APP>
  ```

  `3.11.1` is the version used by your application (as a dependency of
  `react-native-onesignal` npm package):

  ```groovy
  // node_modules/react-native-onesignal/android/build.gradle

  dependencies {
      // ...

      implementation('com.onesignal:OneSignal:3.11.1') {
          exclude group: 'com.android.support'
      }
  }
  ```

  though not strictly necessary, it's better to stick to the same version of
  `OneSignal-Android-SDK` plugin in your fork since it must have been already
  tested against `react-native-onesignal` npm package.

- change `BASE_URL` in _OneSignalRestClient.java_

  ```diff
    // onesignal/src/main/java/com/onesignal/OneSignalRestClient.java

  - private static final String BASE_URL = "https://onesignal.com/api/v1/";
  + private static final String BASE_URL = "https://<PROXY_HOST>/onesignal/api/v1/";
  ```

- build Android library

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
  $ chmod +x gradlew
  $ ./gradlew build -x test
  ```

  output file: _OneSignalSDK/onesignal/build/outputs/aar/onesignal-release.aar_
  (we don't need other files).

## OneSignal-iOS-SDK

- fork and clone <https://github.com/OneSignal/OneSignal-iOS-SDK>

  ```sh
  $ git clone git@github.com:<MY_COMPANY>/OneSignal-iOS-SDK.git
  $ cd OneSignal-iOS-SDK
  ```

- change `SERVER_URL` in _OneSignalCommonDefines.h_

  ```diff
    // iOS_SDK/OneSignalSDK/Source/OneSignalCommonDefines.h

  - #define SERVER_URL @"https://onesignal.com/"
  + #define SERVER_URL @"https://<PROXY_HOST>/onesignal/"
  ```

- build static library

  1. <https://stackoverflow.com/a/22556684/3632318>

  - open _iOS_SDK/OneSignalSDK/OneSignal.xcodeproj_ in Xcode
  - select scheme and destination

    1. <https://github.com/OneSignal/OneSignal-iOS-SDK/commit/ad7ce9f9ee1600609fb8560e6a857f2c361d572a>
    2. <https://stackoverflow.com/a/31181280/3632318>

    | Xcode                                                          |
    | -------------------------------------------------------------- |
    | `Product` (top menu) → `Scheme` → `OneSignal-Static-Framework` |
    | `Product` (top menu) → `Destination` → `Generic iOS Device`    |

    it's necessary to build library not only for `armv7` and `arm64` but for
    `i386` and `x86_64` architectures as well because actual device uses ARM
    processor while simulator uses Intel processor (your host machine):

    > <https://www.quora.com/Why-do-I-need-x86_64-architecture-under-xcode-while-compiling-an-iOS-app/answers/24349415>
    >
    > You will build for the following architectures:
    >
    > - armv7 - for 32-bit devices
    > - arm64 - for 64-bit devices
    > - i386 - for simulators of 32-bit devices
    > - x86_64 - for simulators of 64-bit devices
    >
    > The simulators run code natively on the computer so they are x86. You will
    > only submit armv7 and arm64 to the store.

    in the end your application will fail to build in Xcode if it's linked with
    a static library supporting `armv7` and `arm64` architectures only.

    the problem is that if you select `OneSignal` scheme (and `OneSignal` target
    accordingly), you'll be able to build for `armv7` and `arm64` architectures
    only - manually adding `i386` and `x86_64` architectures in `Build Settings`
    of `OneSignal` target will cause the build of static library to fail.

    but `OneSignal-Static-Framework` scheme runs _build_fat_framework.sh_ script
    which seems to solve this problem by creating a universal binary file:

    ```sh
    # iOS_SDK/OneSignalSDK/build_fat_framework.sh

    # Generates a universal/fat framework that can be used in multiple
    # architectures (x86_64 and arm64) to support both simulator and actual
    # devices.

    # Build x86 based framework to support iOS simulator
    # ...

    # Build arm based framework to support actual iOS devices
    # ...

    # Combine results
    # use lipo to combine device & simulator binaries into one
    # ...
    ```

  output file: _iOS_SDK/OneSignalSDK/Framework/OneSignal.framework/OneSignal_ -
  (it's a symlink which points to a static library).

  use `lipo` to check a static library contains all required architectures:

  ```sh
  $ lipo -archs OneSignal
  armv7 i386 x86_64 arm64
  ```

## react-native-onesignal

- fork and clone <https://github.com/geektimecoil/react-native-onesignal>

  ```sh
  $ git clone git@github.com:<MY_COMPANY>/react-native-onesignal.git
  $ cd react-native-onesignal
  $ git checkout 3.3.1
  $ git checkout -b <MY_APP>
  ```

  `3.3.1` is the version used by your application:

  ```conf
  # yarn.lock

  "react-native-onesignal@github:<MY_COMPANY>/react-native-onesignal#iceperk":
    version "3.3.1"
  ```

  though not strictly necessary, it's better to stick to the same version of
  `react-native-onesignal` npm package since you've been already using it in
  your application.

- update Android SDK dependency (using JAR)

  NOTE: this method of updating Android SDK dependency is not recommended since
  eventually it causes application to crash on old Android devices (Android 6-)
  when the latter goes to the background - this error is sent to Sentry:

  ```
  java.lang.IllegalArgumentException: No such service ComponentInfo{com.iceperkapp/com.onesignal.SyncJobService}
  ```

  => update Android SDK dependency using AAR instead (see the next section).

  input: new Android library file (_onesignal-release.aar_).

  - extract _classes.jar_ from _onesignal-release.aar_

    > <https://stackoverflow.com/a/21485222/3632318>
    >
    > The AAR file consists of a JAR file and some resource files

  - rename _classes.jar_ to _onesignal-\<MY_APP>.jar_ (or whatever)

    even though the name of JAR file can be anything, don't forget to keep it
    in sync with a reference to this file in _android/build.gradle_.

  - copy _onesignal-\<MY_APP>.jar_ to _android/libs/_
  - replace original Android SDK dependency in _android/build.gradle_

    1. <https://developer.android.com/studio/build/dependencies>

    NOTE: make sure to remove the block with `exclude group` line - OneSignal
    classes are not found otherwise.

    ```diff
      // android/build.gradle

      dependencies {
          // ...

    -     implementation('com.onesignal:OneSignal:3.11.1') {
    -         // Exclude com.android.support(Android Support library) as the version range starts at 26.0.0
    -         //    This is due to compileSdkVersion defaulting to 23 which cant' be lower than the support library version
    -         //    And the fact that the default root project is missing the Google Maven repo required to pull down 26.0.0+
    -         exclude group: 'com.android.support'
    -         // Keeping com.google.android.gms(Google Play services library) as this version range starts at 10.2.1
    -     }
    +     implementation files('libs/onesignal-<MY_APP>.jar')
    ```

- _[RECOMMENDED]_ update Android SDK dependency (using AAR)

  input: new Android library file (_onesignal-release.aar_).

  - rename _onesignal-release.aar_ to _onesignal-\<MY_APP>.aar_

    the name of AAR can be anything and it's not referenced anywhere directly
    including _android/build.gradle_.

  - copy _onesignal-\<MY_APP>.jar_ to _android/libs/_
  - replace original Android SDK dependency in _android/build.gradle_

    1. <https://developer.android.com/studio/build/dependencies>
    2. <https://life.nimbco.com/2013/09/referencing-local-aar-files-with-android-studios-new-gradle-based-build-system/>

    NOTE: make sure to remove the block with `exclude group` line - OneSignal
    classes are not found otherwise.

    ```diff
      // android/build.gradle

    + repositories {
    +     mavenCentral()
    +     flatDir {
    +         dirs 'libs'
    +     }
    + }
    +
      dependencies {
          // ...

    -     implementation('com.onesignal:OneSignal:3.11.1') {
    -         // Exclude com.android.support(Android Support library) as the version range starts at 26.0.0
    -         //    This is due to compileSdkVersion defaulting to 23 which cant' be lower than the support library version
    -         //    And the fact that the default root project is missing the Google Maven repo required to pull down 26.0.0+
    -         exclude group: 'com.android.support'
    -         // Keeping com.google.android.gms(Google Play services library) as this version range starts at 10.2.1
    -     }
    +     implementation 'com.onesignal:OneSignal:3.11.1@aar'
      }
    ```

- update iOS SDK dependency

  input: new static library file (_OneSignal_).

  - rename _OneSignal_ to _libOneSignal.a_
  - copy _libOneSignal.a_ to _ios/_ replacing existing file with the same name

  don't forget to link this library (_libRCTOneSignal.a_ to be precise - the
  same library but without support for `i386` and `x86_64` architectures) to
  both your application and OneSignal notification service extension.

- commit and push changes

## your application

- use fork of `react-native-onesignal` npm package

  ```diff
    // package.json

  - "react-native-onesignal": "^3.2.14",
  + "react-native-onesignal": "github:<MY_COMPANY>/react-native-onesignal#<MY_APP>",
  ```

  ```sh
  $ yarn
  ```

- build and run application in emulator

  ```sh
  $ react-native run-ios
  $ react-native run-android
  ```
