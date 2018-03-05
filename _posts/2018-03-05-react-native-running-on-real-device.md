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

application:

- make sure notebook and real device use the same Wi-Fi network
- open _\<project_name>.xcworkspace_
- select real device as the build target in Xcode toolbar
- change server IP address for development environment

  now server is available by notebook's local IP address - not `localhost`.

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
- hot reloading doesn't work - enable live reload instead
- shake real device to access Developer Menu
