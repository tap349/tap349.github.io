---
layout: post
title: React Native - Running on Real Device
date: 2018-03-05 14:00:08 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://facebook.github.io/react-native/docs/running-on-device.html>

notes for both iOS and Android:

- hot reloading works as a rule (use live reload if it doesn't)
- shake real device to access Developer Menu
- select `Debug JS Remotely` in Developer Menu to open React Native Debugger

iOS
---

- make sure notebook and real device use the same Wi-Fi network
- open _\<project_name>.xcworkspace_
- select real device as the build target in Xcode toolbar
- connect to development server

  change server IP address for development environment: development
  server is available by notebook's local IP address - not `localhost`.

  _app/api/ApiHelpers.js_:

  ```diff
    const LOCALHOST = Platform.OS === 'ios'
  -   ? 'http://localhost:3000'
  +   ? 'http://192.168.0.36:3000'
      : 'http://10.0.2.2:3000';
  ```

  _app/api/graphql/run.js_:

  ```diff
    const DEVELOPMENT_BASE_URL = Platform.OS === 'ios'
  -   ? 'http://localhost:3000'
  +   ? 'http://192.168.0.36:3000'
      : 'http://10.0.2.2:3000';
  ```

- build and run application

Android
-------

- uninstall application from device if it's already installed from Google Play

  or else you'll get `INSTALL_FAILED_DUPLICATE_PERMISSION` error:

  ```
  $ react-native run-android
  ...
  Installing APK 'app-debug.apk' on 'GT-I9500 - 5.0.1' for app:debug
  04:27:39 E/SplitApkInstaller: Failed to finalize session : INSTALL_FAILED_DUPLICATE_PERMISSION:
    Package com.iceperkapp attempting to redeclare permission com.iceperkapp.permission.C2D_MESSAGE already owned by com.iceperkapp
  ```

- enable `USB debugging` and plug in real device via USB
- make sure real device is listed in `adb devices`

  ```
  $ adb devices
  List of devices attached
  emulator-5554 device
  4d00af1d6fa7306d device
  ```

- connect to development server

  ```
  $ adb reverse tcp:8081 tcp:8081
  ```

  NOTE: maybe it's not required since:

  ```
  $ react-native run-android
  ...
  Running /usr/local/share/android-sdk/platform-tools/adb -s 4d00af1d6fa7306d reverse tcp:8081 tcp:8081
  Starting the app on 4d00af1d6fa7306d (/usr/local/share/android-sdk/platform-tools/adb -s 4d00af1d6fa7306d shell am start -n com.iceperkapp/com.iceperkapp.MainActivity)...
  ```

- build and run application

  running `react-native run-android` will install application on all
  devices listed in `adb devices` output - say, both real device and
  emulator (it's safe to close emulator).
