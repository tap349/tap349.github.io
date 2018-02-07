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

  _ios/iceperkapp/Info.plist_ (Xcode: `General` -> `Identity` -> `Build`):

  ```xml
  <key>CFBundleShortVersionString</key>
  <string>3.11.2</string>
  <!--...-->
  <key>CFBundleVersion</key>
  <string>68</string>
  ```

  _android/app/build.gradle_:

  ```groovy
  versionCode 68
  versionName "3.11.2"
  ```

iOS
---

- open _iceperkapp.xcworkspace_ in Xcode
  - select `Generic iOS Device`
  - create archive: `Product` (top menu) -\> `Archive`
  - click `Upload to App Store..` button when archive is created
  - follow on-screen instructions (leave defaults)
  - click `Upload` button in the end
- [TestFlight] open `iTunes Connect` in browser
  - go to `TestFlight` tab
  - go to `BUILDS` (left menu) -\> `iOS`
  - wait till new build is processed (it might temporarily disappear from the list)
  - open just uploaded iOS build
  - click `Provide Export Compliance Information` button
  - select `No` (app doesn't use encryption) in pop-up window
  - click `Start Internal Testing` button
- [App Store] open `iTunes Connect` in browser
  - go to `App Store` tab
  - click `⨁ VERSION OR PLATFORM` link -> `iOS` (pop-up menu)
  - fill `What's New in This Version` section
  - select new build
  - select `Automatically release this version` option
  - click `Save` button
  - click `Submit for Review` button
  - select `No` (app doesn't use IDFA) on `Advertising Identifier` page
  - click `Submit` button

Android
-------

copy _gradle.properties_ and _iceperkkeystore.keystore_ (release store file)
from `Complead/iceperkapp_certificates/android` GitHub repo (see _README.md_)
before building releases.

- assemble release

  ```sh
  $ build_android_release='cd android && ./gradlew assembleRelease; cd ..'
  $ build_android_release
  ```

  new release is saved as _android/app/build/outputs/apk/app-release.apk_.

- upload release to Google Drive: `Shared with me` -> `ICEperk` -> `AppBuilds`

  replace existing `app-release.apk` file with a new one.

- open `Google Play Console` in browser
  - open `Release management` (left menu) -> `App releases`
  - click `MANAGE PRODUCTION` button in `Production` section
  - click `CREATE RELEASE` button
  - select APK to add from filesystem
  - fill `What's new in this release?` section
  - click `SAVE` button
  - click `REVIEW` button

iceperk
-------

- increment build numbers in `Api::V1::UserUpdatedAtsController` and deploy
  `iceperk` application *after* new `iceperkapp` releases appear in *both*
  App Store and Google Play

  this is to notify all users about a new release via modal window inside
  application (optional).

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

### [Android] ENOTEMPTY: directory not empty

build failed:

{% raw %}
```sh
$ cd android && ./gradlew assembleRelease && cd ..
...
:app:bundleReleaseJsAndAssets
Scanning 773 folders for symlinks in <app_dir>/node_modules (41ms)
Scanning 773 folders for symlinks in <app_dir>/node_modules (37ms)
Loading dependency graph, done.

ENOTEMPTY: directory not empty, rmdir '/var/folders/1g/28pm7dxn0_l569vjyzlzp2zw0000gn/T/react-native-packager-cache-f18bd0fb39fa7507ecdd2fb6cd91757d41b78c44/cache'

:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssets'.
> Process 'command 'node'' finished with non-zero exit value 1
```
{% endraw %}

**solution**

reset cache:

```sh
$ yarn start --reset-cache
```
