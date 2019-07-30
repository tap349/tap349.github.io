---
layout: post
title: React Native - Proxy for Onesignal
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

say, proxy URL is `https://example.com`.

## fork and modify SDK's

### Android

- fork Android SDK
- edit `BASE_URL` in `OneSignalRestClient` class

  ```diff
    // onesignal/src/main/java/com/onesignal/OneSignalRestClient.java

  - private static final String BASE_URL = "https://onesignal.com/api/v1/";
  + private static final String BASE_URL = "https://example.com/onesignal/api/v1/";
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

