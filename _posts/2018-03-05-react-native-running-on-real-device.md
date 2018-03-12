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

  change server address for development environment: development server
  is now available by notebook's local IP address - not `localhost`.

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
- make sure real device is listed in `adb devices` output

  ```
  $ adb devices
  List of devices attached
  emulator-5554 device
  4d00af1d6fa7306d device
  ```

- connect to development server

  1. <https://stackoverflow.com/a/43277765/3632318>

  forward port (Android 5.0+ only):

  ```
  $ adb reverse tcp:3000 tcp:3000
  ```

  now local `3000` port is mapped to mobile's `3000` port.

  NOTE: `3000` is the port development server is listening on
        (accordingly it's specified as development server port in
        both _app/api/ApiHelpers.js_ and _app/api/graphql/run.js_).

  also change server address for development environment: development
  server is now available by both `localhost` and notebook's local IP
  address (but not `10.0.2.2` - like in case of emulator).

  _app/api/ApiHelpers.js_:

  ```diff
    const LOCALHOST = Platform.OS === 'ios'
      ? 'http://localhost:3000'
  -   ? 'http://10.0.2.2:3000'
  +   : 'http://localhost:3000';
  ```

  _app/api/graphql/run.js_:

  ```diff
    const DEVELOPMENT_BASE_URL = Platform.OS === 'ios'
      ? 'http://localhost:3000'
  -   ? 'http://10.0.2.2:3000'
  +   : 'http://localhost:3000';
  ```

- build and run application

  running `react-native run-android` will install application on all
  devices listed in `adb devices` output - say, both real device and
  emulator (it's safe to close emulator).
