---
layout: post
title: Iceperk - Releases
date: 2017-12-26 17:52:35 +0300
access: private
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>AD</dt>
  <dd>Apple Developer</dd>

  <dt>GPC</dt>
  <dd>Google Play Console</dd>

  <dt>IC</dt>
  <dd>iTunes Connect</dd>
</dl>

<hr>

say, new release branch name is `release_3_16`.

GitHub (part 1): merge iceperk release branch into master branch
----------------------------------------------------------------

NOTE: branches can be merged manually or via PR.

- [`iceperk`] merge `release_3_16` branch into `develop` branch
  - PR name: `Develop (Release 3.16)`
- [`iceperk`] merge `develop` branch into `master` branch
  - PR name: `Master (Release 3.16)`
- [`iceperkapp`] don't merge `release_3_16` branch into `develop` branch so far!

test backend and new release
----------------------------

seed data for testing:

- phone numbers: +7 900 000-00-01 (and so forth)
- profile names: beta0001
- team names: beta0001 team

### test new backend against old release (sanity checks)

this is to make sure nothing is broken for old releases.

- [`iceperkapp`] switch to `develop` branch (previous release)
- [`iceperkapp`] change environment to `development` in _Env.js_
- [`iceperk`] switch to `master` branch
- [`iceperk`] switch to production billing in _billing.yml_
- use emulator and development server to run all sanity checks
- [`iceperk`] deploy `master` branch to production

### test new release against new backend (new features)

- [`iceperkapp`] switch to `release_3_16` branch (new release)
- [`iceperkapp`] change environment to `development` in _Env.js_
- [`iceperk`] switch to `master` branch
- [`iceperk`] switch to production billing in _billing.yml_ (if necessary)
- use emulator and development server to test new features from Trello

### test new release against production server (sanity checks)

- [`iceperkapp`] switch to `release_3_16` branch (new release)
- [`iceperkapp`] change environment to `production` in _Env.js_
- [`iceperk`] production server is used now
- use emulator and production server to run all sanity checks

or else it's possible to run sanity checks on real device instead of emulator
(using a test build).

GitHub (part 2): merge iceperkapp release branch into develop branch
--------------------------------------------------------------------

- [`iceperkapp`] merge `release_3_16` branch into `develop` branch
  - PR name: `Develop (Release 3.16)`
- [`iceperkapp`] switch to `develop` branch (we use it to build releases)

prepare new release in iceperkapp
---------------------------------

- change environment to `production` in _Env.js_ before generating releases
- increment version number and build number in both iOS and Android projects

  NOTE: these numbers might have been already changed right after creating
        `release_3_16` branch.

  _ios/iceperkapp/Info.plist_ (Xcode: `General` → `Identity` → `Build`):

  ```xml
  <key>CFBundleShortVersionString</key>
  <string>3.16</string>
  <!-- ... -->
  <key>CFBundleVersion</key>
  <string>96</string>
  ```

  don't forget to bump both numbers in _Info.plist_ files of all extensions
  your iOS application contains (say, `OneSignalNotificationServiceExtension`)
  or else you'll get `ITMS-90473` warnings (`CFBundleVersion Mismatch` and
  `CFBundleShortVersionString Mismatch`) when uploading archive to App Store.

  _android/app/build.gradle_:

  ```groovy
  versionCode 96
  versionName "3.16"
  ```

- commit changes and push to `develop` branch

build and publish new release
-----------------------------

### iOS

- open _iceperkapp.xcworkspace_ in Xcode
  - select `Generic iOS Device`
  - create archive: `Product` (top menu) → `Archive`
  - click `Distribute App` button when archive is created
  - walk through the steps of the wizard

    > Select a method of distribution:

    - [x] `iOS App Store`

    > Select a destination:

    - [x] `Upload`

    > App Store distribution options:

    - [x] `Include bitcode for iOS content`
    - [x] `Upload your app's symbols to receive symbolicated reports from Apple`

    > Re-sign "iceperkapp":

    - [x] `Automatically manage signing`

    > Create a new iOS Distribution certificate:

    NOTE: this step is missing if existing iOS Distribution certificate has
          not expired.

    - [x] `Generate an iOS Distribution certificate`

    iOS Distribution signing certificate for your company will be generated,
    added to your local Keychain and uploaded to:

    | AD: `Certificates, IDs & Profiles` (left menu)
    | `Certificates` (left menu)

    > Export signing certificate:

    NOTE: this step is missing if iOS Distribution certificate wasn't created
          in a previous step.

    - `Export Signing Certificate...` (button)

  - click `Upload` button in the end
- [TestFlight] open `iTunes Connect` in browser
  - go to `TestFlight` tab
  - go to `BUILDS` (left menu) → `iOS`
  - wait till new build is processed (it might temporarily disappear from the list)
  - open new iOS build
  - click `Provide Export Compliance Information` button
  - select `No` (app doesn't use encryption) in popup window
  - click `Start Internal Testing` button
  - install new build via TestFlight application and run selective checks
- [App Store] open `iTunes Connect` in browser
  - go to `App Store` tab
  - click `⨁ VERSION OR PLATFORM` link → `iOS` (popup menu)
  - enter new `Store Version Number` (say, 3.16) in popup window
  - fill `What's New in This Version` section
  - screenshots must be added for iPhone 6.5" and iPhone 5.5" Displays
  - select new build
  - select `Automatically release this version` option
  - click `Save` button
  - click `Submit for Review` button
  - select `Yes` (app uses IDFA now) on `Advertising Identifier` page
    - [x] `Serve advertisements within the app`
  - click `Submit` button

### Android

copy _gradle.properties_ and _iceperkkeystore.keystore_ (release store file)
from `C******d/iceperkapp_certificates/android` GitHub repo (see _README.md_)
before building releases.

- assemble release

  ```sh
  $ build_android_release='cd android && ./gradlew assembleRelease; cd ..'
  $ build_android_release
  ```

  release is saved as _android/app/build/outputs/apk/app-release.apk_.

- upload release to Google Drive

  | Google Drive: `Shared with me` → `ICEperk` → `AppBuilds`

  - replace existing `app-release.apk` file with a new one

- install release on any Android device and run selective checks
- publish release in Google Play Console

  | GPC: `All applications` → `Хоккей. Ледовое братство`
  | `Release management` (left menu) → `App releases` → `Production track` (section) → `MANAGE` (button) → `CREATE RELEASE` (button)

  > New release to production

  - `BROWSE_FILES` (button) → add new APK
  - `Release name` (input): `3.16` (for example)
  - `What's new in this release?` (textarea): release notes
  - `SAVE` (button) → `REVIEW` (button)

  > Confirm rollout to production: 3.16

  - `Rollout percentage` (input): `100%` (it's `50%` by default)
  - `START ROLLOUT TO PRODUCTION` (button)

GitHub (part 3): create iceperkapp release on GitHub
----------------------------------------------------

this is done AFTER release is published in both stores and all seems to be
working okay:

- [`iceperkapp`] merge `develop` branch into `master` branch
  - PR name: `Master (Release 3.16)`
- [`iceperkapp`] create new release on GitHub

  | GitHub: `37 releases` (link in repo header) → `Draft a new release` (button) → `Releases` (tab)

  - `Tag version` (input + combobox): `3.16`
  - `Target` (combobox): `master`
  - `Release title` (input): `Release 3.16`
  - `Publish release` (button)

- [`iceperk`] create and push branch for new release from `develop` branch
  (say, `release_3_17`)
- [`iceperkapp`] create and push branch for new release from `develop` branch
  (say, `release_3_17`)

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
- disable bitcode in `Build Settings` → `Build Options` → `Enable Bitcode`

### [iOS] "\<Company>" has one iOS Distribution but its private key is not installed

the error occurs when trying to upload archive to App Store.

**solution**

- download iOS certificate for `<Company>` (`iOS Distribution` Type)

  1. <https://developer.apple.com/account/resources/certificates>

  use this link to download certificate (`.cer` file) and import it into the
  login keychain (its name is `iPhone Distribution: <Company>`).

  NOTE: this certificate might have been already imported.

- get private key of iOS Distribution certificate

  get private key (`.p12` file) from its creator and import it into the login
  keychain (its name is like `iOS Distribution: <Company>`).

### [Android] ENOTEMPTY: directory not empty

build failed:

```
$ cd android
$ ./gradlew assembleRelease
...
:app:bundleReleaseJsAndAssets
Scanning 773 folders for symlinks in <APP_DIR>/node_modules (41ms)
Scanning 773 folders for symlinks in <APP_DIR>/node_modules (37ms)
Loading dependency graph, done.

ENOTEMPTY: directory not empty, rmdir '/var/folders/1g/28pm7dxn0_l569vjyzlzp2zw0000gn/T/react-native-packager-cache-f18bd0fb39fa7507ecdd2fb6cd91757d41b78c44/cache'

:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssets'.
> Process 'command 'node'' finished with non-zero exit value 1
```

**solution**

reset cache:

```sh
$ react-native start --reset-cache
```

### [iOS] uploaded build doesn't appear on `TestFlight` tab

it's okay for build to disappear temporarily from the list of builds but
sometimes it might not appear in the list again - no matter how long you
wait.

**solution**

bump build number and upload new build.

### [iOS] test build doesn't appear in TestFlight application

build has `Testing` status for `App Store Connect Users` but TestFlight
application still shows `No Apps Available to Test` message.

**solution**

bump build number and upload new build.

if it doesn't help:

- remove user (say, yourself) from testers and then add him back

  | IC: `My Apps` → `<MY_APP>` → `TestFlight` (tab)
  | `App Store Connect Users` (left menu) → `Testers` (tab)

  invitation to test new build will be sent to this user by email.

- follow invitation link from email (`View in TestFlight` button)
- follow on-screen instructions to accept invitation

  > To accept this invitation:
  >
  > 1. Get TestFlight from the App Store.
  > 2. Open TestFlight and choose Redeem.
  > 3. Enter \<INVITATION_CODE> and start testing.

### [iOS] iTunes Store Operation Failed

the error occurs when trying to upload archive to App Store in Xcode:

```
iTunes Store Operation Failed

The status is FAILED for the file named /var/folders/<...> and the error
description is 'Destination: Disk quota exceeded (5)'
```

**solution**

1. <https://stackoverflow.com/a/49045515/3632318>

I bumped build number but forgot to change version number (`CFBundleVersion`
and `CFBundleShortVersionString` properties in _Info.plist_ accordingly).

it might also help to validate archive before uploading it to App Store
(`Validate...` button) - this gives more meaningful error messages when
archive is not valid.

### GitHub release is created with a wrong target (branch)

if you have accidentally published release with a wrong target (branch), you
can't edit target afterwards (though you can edit, say, tag or release title).

follow these steps in this case to create new release with the same tag
but different target:

- delete release

  | GitHub: `37 releases` (link in repo header) → `Release 3.16` (link) → `Delete` (button)

- delete current tag

  1. <https://gist.github.com/mobilemind/7883996>

  you won't be able to create new release with this tag but different target
  unless you remove this tag from GitHub.

  fetch changes (including new tag) from GitHub:

  ```sh
  $ git up
  ```

  delete local tag:

  ```sh
  $ git tag -d 3.16
  ```

  delete remote tag:

  ```sh
  $ git push origin :refs/tags/3.16
  / or
  $ git push --delete origin 3.16
  ```

### [iOS] Multiple commands produce '...libyoga.a'

build failed in Xcode:

```
Multiple commands produce '...libyoga.a':
1) Target 'yoga' has a command with output '...libyoga.a'
2) Target 'yoga' has a command with output '...libyoga.a'

Multiple commands produce '...libReact.a':
1) Target 'React' has a command with output '...libReact.a'
2) Target 'React' has a command with output '...libReact.a'
```

**solution**

1. <https://github.com/facebook/react-native/issues/20492#issuecomment-422958184>

> <https://github.com/facebook/react-native/issues/20492#issuecomment-422958184>
>
> The following is needed to ensure the "archive" step works in XCode. It
> removes React & Yoga from the Pods project, as it is already included in
> the main project.
>
> Without this, you'd see errors when you archive like:
> "Multiple commands produce ... libReact.a"
> "Multiple commands produce ... libyoga.a"

```ruby
# ios/Podfile

post_install do |installer|
  installer.pods_project.targets.each do |target|
    targets_to_ignore = %w[React yoga]

    if targets_to_ignore.include?(target.name)
      target.remove_from_project
    end
  end
end
```

```sh
$ cd ios
$ pod install
```

### resource android:style/TextAppearance.Material.Widget.Button.Colored not found

```
$ cd android
$ ./gradlew assembleRelease
...
> Task :react-native-exit-app:verifyReleaseResources FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':react-native-exit-app:verifyReleaseResources'.
> 1 exception was raised by workers:
  com.android.builder.internal.aapt.v2.Aapt2Exception: Android resource linking failed
  error: resource android:style/TextAppearance.Material.Widget.Button.Borderless.Colored not found.
  error: resource android:style/TextAppearance.Material.Widget.Button.Colored not found.
  <APP_DIR>/node_modules/react-native-exit-app/android/build/intermediates/res/merged/release/values-v26/values-v26.xml:7: error: resource android:attr/colorError not found.
```

the error is usually accompanied by this warning when configuring corresponding
project:

```
$ cd android
$ ./gradlew assembleRelease
...
> Configure project :react-native-exit-app
WARNING: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'.
It will be removed at the end of 2018. For more information see: http://d.android.com/r/tools/update-dependency-configurations.html
WARNING: The specified Android SDK Build Tools version (25.0.0) is ignored, as it is below the minimum supported version (28.0.3) for Android Gradle Plugin 3.4.0.
Android SDK Build Tools 28.0.3 will be used.
To suppress this warning, remove "buildToolsVersion '25.0.0'" from your build.gradle file, as each version of the Android Gradle Plugin now has a default version of the build tools.
```

**solution**

explanation:

> <https://stackoverflow.com/a/49332191/3632318>
>
> android:style/TextAppearance.Material.Widget.Button.Borderless.Colored was
> added in API 24 so you can't use it with version 23. You can use a style that
> was added before version 23.

in brief RN package is using component which is not available in API level 23
(this API level is specified in _android/build.gradle_ file of that RN package).

=> increase API level supported by RN package (either fork RN packaged yourself
or find a fork where it has been already done):

```diff
  // android/build.gradle

  android {
-   compileSdkVersion 23
-   buildToolsVersion "23.0.1"
+   compileSdkVersion 28
+   buildToolsVersion "28.0.1"

    defaultConfig {
        minSdkVersion 16
-       targetSdkVersion 22
+       targetSdkVersion 28
    }
```

> <https://stackoverflow.com/a/24523113/3632318>
>
> compileSdkVersion is the API version of Android that you compile against.
>
> buildToolsVersion is the version of the compilers (aapt, dx, renderscript
> compiler, etc...) that you want to use. For each API level (starting with 18),
> there is a matching .0.0 version.

> <https://developer.android.com/studio/releases/gradle-plugin.html#3-2-0>
>
> 3.2.1 (October 2018)
>
> With this update, you no longer need to specify a version for the SDK Build
> Tools. The Android Gradle plugin now uses version 28.0.3 by default.
