---
layout: post
title: React Native - Generating release
date: 2017-12-26 17:52:35 +0300
access: private
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

iceperkapp
-----------

- change environment to `production` in _Env.js_ before generating releases
- increment build number in iOS and Android projects

  _ios/iceperkapp/Info.plist_
  (or `General` -> `Identity` -> `Build` in Xcode):

  ```xml
  <key>CFBundleVersion</key>
  <string>58</string>
  ```

  _android/app/build.gradle_:

  ```groovy
  versionCode 58
  ```

iOS
---

- open _iceperkapp.xcworkspace_ in Xcode
- select `Generic iOS Device`
- create archive: `Product` (top menu) -> `Archive`
- click `Upload to App Store..` button when archive is created
- follow on-screen instructions (leave defaults)
- click `Upload` button in the end

Android
-------

- copy _gradle.properties_ and _iceperkkeystore.keystore_ (release store file)
  from `Complead/iceperkapp_certificates/android` GitHub repo (see _README.md_)
- assemble release

  ```sh
  $ cd android && ./gradlew assembleRelease && cd ..
  ```

  new release is saved as _android/app/build/outputs/apk/app-release.apk_.

iceperk
-------

- increment build numbers in `Api::V1::UserUpdatedAtsController` and deploy
  `iceperk` application *after* new `iceperkapp` releases appear in *both*
  App Store and Google Play

  this is required to show a notification message about a new release to all
  users (modal window inside application).

troubleshooting
---------------

### [iOS] Invalid bitcode version (Producer: '802.0.41.0_0' Reader: '800.0.42.1_0')

build failed in Xcode:

```
error: Invalid bitcode version (Producer: '802.0.42.0_0' Reader: '800.0.42.1_0')
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://stackoverflow.com/questions/43480556/xcode-8-2-1-error-invalid-bitcode-version-producer-802-0-41-0-0-reader>

either:

- *[RECOMMENDED]* update Xcode to 8.3
- disable bitcode in `Build Settings` -> `Build Options` -> `Enable Bitcode`

### [iOS] "<Company>" has one iOS Distribution but its private key is not installed

the error occurs when trying to upload archive to App Store.

**solution**

- download iOS certificate for `<Company>` (`iOS Distribution` Type) from
  [iOS Certificates](https://developer.apple.com/account/ios/certificate/)
  (_ios\_distribution.cer_ file) and import it into the login keychain
  (its name is something like `iPhone Distribution: <Company>`)

  NOTE: this certificate might have been imported already

- get private key of iOS Distribution certificate
  (_iOS Distribution\_<Company>.p12_ file) from its creator and import it into
  the login keychain (its name is something like `iOS Distribution: <Company>`)
