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
  <!--...-->
  <key>CFBundleVersion</key>
  <string>96</string>
  ```

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
  - create archive: `Product` (top menu) -\> `Archive`
  - click `Upload to App Store..` button when archive is created
  - follow on-screen instructions (leave defaults)
  - click `Upload` button in the end
- [TestFlight] open `iTunes Connect` in browser
  - go to `TestFlight` tab
  - go to `BUILDS` (left menu) -\> `iOS`
  - wait till new build is processed (it might temporarily disappear from the list)
  - open new iOS build
  - click `Provide Export Compliance Information` button
  - select `No` (app doesn't use encryption) in popup window
  - click `Start Internal Testing` button
  - run selective checks
- [App Store] open `iTunes Connect` in browser
  - go to `App Store` tab
  - click `⨁ VERSION OR PLATFORM` link → `iOS` (popup menu)
  - enter new `Store Version Number` (say, 3.16) in popup window
  - fill `What's New in This Version` section
  - screenshots are added for iPhone 5.5" Display only
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

  new release is saved as _android/app/build/outputs/apk/app-release.apk_.

- upload release to Google Drive

  | Google Drive: `Shared with me` → `ICEperk` → `AppBuilds`

  - replace existing `app-release.apk` file with a new one

- run selective checks
- publish release in Google Play Console

  | GPC: `All applications` → `Хоккей. Ледовое братство`
  | `Release management` (left menu) → `App releases` → `MANAGE PRODUCTION` (button) → `CREATE RELEASE` (button)

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

this is done AFTER release is published in both stores and all seems
to be working okay:

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

### [iOS] "<Company>" has one iOS Distribution but its private key is not installed

the error occurs when trying to upload archive to App Store.

**solution**

- download iOS certificate for `<Company>` (`iOS Distribution` Type) from
  [iOS Certificates](https://developer.apple.com/account/ios/certificate/)
  (_ios\_distribution.cer_ file) and import it into the login keychain
  (its name is something like `iPhone Distribution: <Company>`)

  NOTE: this certificate might have been imported already

- get private key of iOS Distribution certificate
  (_iOS Distribution\_\<Company>.p12_ file) from its creator and import it into
  the login keychain (its name is something like `iOS Distribution: <Company>`)

### [Android] ENOTEMPTY: directory not empty

build failed:

{% raw %}
```sh
$ cd android && ./gradlew assembleRelease && cd ..
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
{% endraw %}

**solution**

reset cache:

```sh
$ react-native start --reset-cache
```

### [iOS] uploaded build doesn't appear on `TestFlight` tab

it's okay for build to disappear temporarily from the list of builds
but sometimes it might not appear in the list again - no matter how
long you wait.

**solution**

bump build number and upload new build.

### [iOS] iTunes Store Operation Failed

the error occurs when trying to upload archive to App Store:

```
iTunes Store Operation Failed

The status is FAILED for the file named /var/folders/<...> and the error
description is 'Destination: Disk quota exceeded (5)'
```

**solution**

1. <https://stackoverflow.com/a/49045515/3632318>

I bumped build number but forgot to change version number as well
(`CFBundleVersion` and `CFBundleShortVersionString` properties in
_Info.plist_ accordingly).

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
