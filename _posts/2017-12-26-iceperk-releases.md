---
layout: post
title: Iceperk - Releases
date: 2017-12-26 17:52:35 +0300
access: private
comments: true
categories: [react-native]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
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

## GitHub (part 1): merge iceperk release branch into master branch

NOTE: branches can be merged manually or via PR.

- [`iceperk`] merge `release_3_16` branch into `develop` branch
  - PR name: `Develop (Release 3.16)`
- [`iceperk`] merge `develop` branch into `master` branch
  - PR name: `Master (Release 3.16)`
- [`iceperkapp`] don't merge `release_3_16` branch into `develop` branch so far!

## test backend and new release

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

## GitHub (part 2): merge iceperkapp release branch into develop branch

- [`iceperkapp`] merge `release_3_16` branch into `develop` branch
  - PR name: `Develop (Release 3.16)`
- [`iceperkapp`] switch to `develop` branch (we use it to build releases)

## prepare new release in iceperkapp

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

  don't forget to bump both numbers in _Info.plist_ files of all extensions your
  iOS application contains (say, `OneSignalNotificationServiceExtension`) or
  else you'll get `ITMS-90473` warnings (`CFBundleVersion Mismatch` and
  `CFBundleShortVersionString Mismatch`) when uploading archive to the App
  Store.

  _android/app/build.gradle_:

  ```groovy
  versionCode 96
  versionName "3.16"
  ```

- commit changes and push to `develop` branch

## build and publish new release

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

    NOTE: this step is missing if existing iOS Distribution certificate has not
    expired.

    - [x] `Generate an iOS Distribution certificate`

    iOS Distribution signing certificate for your company will be generated,
    added to your local Keychain and uploaded to:

    <!-- prettier-ignore -->
    | AD: `Certificates, IDs & Profiles` (left menu)
    | `Certificates` (left menu)

    > Export signing certificate:

    NOTE: this step is missing if iOS Distribution certificate wasn't created in
    a previous step.

    - `Export Signing Certificate...` (button)

  - click `Upload` button in the end

- [TestFlight] open `iTunes Connect` in browser
  - go to `TestFlight` tab
  - go to `BUILDS` (left menu) → `iOS`
  - wait till new build is processed (it might temporarily disappear from the
    list)
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

  <!-- prettier-ignore -->
  | Google Drive: `Shared with me` → `ICEperk` → `AppBuilds`

  - replace existing `app-release.apk` file with a new one

- install release on any Android device and run selective checks
- publish release in Google Play Console

  <!-- prettier-ignore -->
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

## GitHub (part 3): create iceperkapp release on GitHub

this is done AFTER release is published in both stores and all seems to be
working okay:

- [`iceperkapp`] merge `develop` branch into `master` branch
  - PR name: `Master (Release 3.16)`
- [`iceperkapp`] create new release on GitHub

  <!-- prettier-ignore -->
  | GitHub: `37 releases` (link in repo header) → `Draft a new release` (button) → `Releases` (tab)

  - `Tag version` (input + combobox): `3.16`
  - `Target` (combobox): `master`
  - `Release title` (input): `Release 3.16`
  - `Publish release` (button)

- [`iceperk`] create and push branch for new release from `develop` branch (say,
  `release_3_17`)
- [`iceperkapp`] create and push branch for new release from `develop` branch
  (say, `release_3_17`)

## troubleshooting

<!-- prettier-ignore -->
1. [React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %})

this section contains errors which occurred when preparing release - see the
linked post for compile-time and runtime errors.

### "\<Company>" has one iOS Distribution but its private key is not installed

the error occurs when trying to upload archive to the App Store.

**solution**

- download iOS certificate for `<Company>` (`iOS Distribution` Type)

  1. <https://developer.apple.com/account/resources/certificates>

  use this link to download certificate (`.cer` file) and import it into the
  login keychain (its name is `iPhone Distribution: <Company>`).

  NOTE: this certificate might have been already imported.

- get private key of iOS Distribution certificate

  get private key (`.p12` file) from its creator and import it into the login
  keychain (its name is like `iOS Distribution: <Company>`).

### uploaded build doesn't appear on `TestFlight` tab

it's okay for build to disappear temporarily from the list of builds but
sometimes it might not appear in the list again - no matter how long you wait.

**solution**

bump build number and upload new build.

### test build doesn't appear in TestFlight application

build has `Testing` status for `App Store Connect Users` but TestFlight
application still shows `No Apps Available to Test` message.

**solution**

bump build number and upload new build.

if it doesn't help:

- remove user (say, yourself) from testers and then add him back

  <!-- prettier-ignore -->
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

### iTunes Store Operation Failed

the error occurs when trying to upload archive to the App Store in Xcode:

```
iTunes Store Operation Failed

The status is FAILED for the file named /var/folders/<...> and the error
description is 'Destination: Disk quota exceeded (5)'
```

**solution**

1. <https://stackoverflow.com/a/49045515/3632318>

I bumped build number but forgot to change version number (`CFBundleVersion` and
`CFBundleShortVersionString` properties in _Info.plist_ accordingly).

it might also help to validate archive before uploading it to the App Store
(`Validate...` button) - this gives more meaningful error messages when archive
is not valid.

### GitHub release is created with a wrong target (branch)

if you have accidentally published release with a wrong target (branch), you
can't edit target afterwards (though you can edit, say, tag or release title).

follow these steps in this case to create new release with the same tag but
different target:

- delete release

  <!-- prettier-ignore -->
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
