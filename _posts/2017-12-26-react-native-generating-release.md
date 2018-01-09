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

iOS
---

- open _iceperkapp.xcworkspace_ in Xcode
- select `Generic iOS Device`
- increment build number:

  Xcode: `General` -> `Identity` -> `Build`

  or else in _ios/iceperkapp/Info.plist_:

  ```xml
  <key>CFBundleVersion</key>
  <string>58</string>
  ```

- create archive: `Product` (top menu) -> `Archive`
- click `Upload to App Store..` button when archive is created
- follow on-screen instructions (leave defaults) and click
  `Upload` button in the end

Android
-------

- increment build number

  _android/app/build.gradle_:

  ```groovy
  versionCode 58
  ```

- copy _gradle.properties_ and _iceperkkeystore.keystore_ (release store file)
  from `Complead/iceperkapp_certificates/android` repo

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

### [iOS] "Company" has one iOS Distribution but its private key is not installed

the error occurs when trying to upload archive to App Store.

**solution**

get private key of iOS Distribution certificate (_p12_ file) from its creator
and import it into the login keychain (via `keychain Access` application).
