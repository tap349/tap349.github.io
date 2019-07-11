---
layout: post
title: React Native - Android
date: 2017-05-24 16:27:38 +0300
access: public
comments: true
categories: [react-native, android]
---

<!-- more -->

* TOC
{:toc}
<hr>

installation
------------

1. <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>

### install Java 8

1. [React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %})

see `Could not determine java version from '9.0.4'` Android error in the linked
post.

```sh
$ brew tap homebrew/cask-versions
$ brew cask install adoptopenjdk8
```

### install Node, Watchman and React Native CLI

1. <https://facebook.github.io/react-native/docs/getting-started.html#installing-dependencies>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

```sh
$ brew install node watchman
$ npm install -g react-native-cli
```

### install Android SDK

```sh
$ brew cask install android-sdk
```

### configure paths

_~/.zshenv_:

```conf
# required by `react-native-cli` to be set
export ANDROID_HOME=/usr/local/share/android-sdk
# `android`
path=($path $ANDROID_HOME/tools)
# `sdkmanager` and `avdmanager`
path=($path $ANDROID_HOME/tools/bin)
# `adb`
path=($path $ANDROID_HOME/platform-tools)
```

### install Android SDK packages

1. <https://developer.android.com/studio/command-line/sdkmanager.html>

`android-sdk` brew formula provides basic tools like `sdkmanager` but required
packages must be installed manually (say, you might need different API level).

API level 23 corresponds to Android 6.0 (Marshmallow).

`sdkmanager` commands:

- list all packages

  list all packages along with outdated packages (installed packages that have
  new version available).

  ```sh
  $ sdkmanager --list
  ```

- install packages

  ```sh
  $ sdkmanager 'PACKAGE_NAME'
  ```

  packages are updated if they are already installed.

- update installed packages

  update all installed packages to their latest versions:

  ```sh
  $ sdkmanager --update
  ```

- uninstall specified packages

  ```sh
  $ sdkmanager --uninstall 'PACKAGE_NAME'
  ```

you might not need all mentioned packages when using Genymotion (say, system
images) but still some of them are required (like `platform-tools` for `adb`):

- Google APIs
- Android SDK Platform 23 (Android 6.0 (Marshmallow))
- Intel x86 Atom_64 System Image
- Google APIs Intel x86 Atom_64 System Image
- Android SDK Build-Tools 23.0.3
- Android SDK Platform-Tools (for `adb`)
- Android Emulator (for `emulator`)

```sh
$ sdkmanager 'add-ons;addon-google_apis-google-23'
$ sdkmanager 'platforms;android-23'
$ sdkmanager 'system-images;android-23;default;x86_64'
$ sdkmanager 'system-images;android-23;google_apis;x86_64'
$ sdkmanager 'build-tools;23.0.3'
$ sdkmanager 'platform-tools'
$ sdkmanager 'emulator'
```

even though _$ANDROID_HOME/tools/_ folder contains `emulator` executable, still
it requires `emulator` package to be installed (or else you'll get the very `Qt
library not found` error).

you can launch Android Emulator via any of these executables:

- _$ANDROID_HOME/tools/emulator_ (I'll use this one for no particular reason)
- _$ANDROID_HOME/emulator/emulator_

#### Intel Hardware Accelerated Execution Manager (HAXM)

1. <https://developer.android.com/studio/run/emulator-acceleration>
2. <https://stackoverflow.com/a/36908316/3632318>

```sh
$ sdkmanager 'extras;intel;Hardware_Accelerated_Execution_Manager'
```

> <https://stackoverflow.com/a/36908316/3632318>
>
> Even though the SDK manager says "Installed" it actually means that the Intel
> HAXM executable was downloaded. You will still need to run the installer from
> the "extras" directory to get it installed.

```sh
$ cd $ANDROID_HOME/extras/intel/Hardware_Accelerated_Execution_Manager
$ sudo ./silent_install.sh
```

check current HAXM version:

```sh
$ $ANDROID_HOME/tools/emulator -accel-check
accel:
0
HAXM version 7.3.2 (4) is installed and usable.
accel
```

### create new AVD (Android Virtual Device)

1. <https://developer.android.com/studio/command-line/avdmanager.html>

```sh
$ avdmanager create --help
$ avdmanager list device
$ avdmanager create avd \
  --force \
  --package 'system-images;android-23;google_apis;x86_64' \
  --name Nexus_5X_API_23_x86_64 \
  --abi 'google_apis/x86_64' \
  --device 9
$ avdmanager list avd
```

### configure emulator to use that AVD

1. <https://stackoverflow.com/questions/42718973>
2. <https://developer.android.com/studio/run/emulator-acceleration.html#command-gpu> (`-gpu` option)
3. <https://stackoverflow.com/questions/42792947> (`-skin` option)
4. <https://developer.android.com/guide/practices/screens_support.html#testing> (available skins)

emulator searches for AVDs in the following directories:

- _$ANDROID_AVD_HOME_
- _$ANDROID_SDK_HOME/avd/_
- _~/.android/avd/_

specify `-gpu` and `-skin` options when starting emulator:

```sh
$ emulator -help
$ emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920
```

- `-gpu host` - enables graphics hardware emulation
- `-skin 1080x1920` - changes screen resolution to 1080x1920
  (by default a very low screen resolution is used)

some skin resolutions have corresponding skin names (see link above).

it's a good idea to create an alias for this command in _~/.zshenv_:

```conf
alias avd='emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920'
```

according to emulator's log there is no need to install HAXM separately (it
appears to have been already enabled):

```
Hax is enabled
Hax ram_size 0x40000000
HAX is working and emulator runs in fast virt mode.
```

running
-------

NOTE: emulator for Android is called Android Emulator (will be referred to as
      just emulator for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS server)

if you don't start it manually it will be started by `react-native run-android`
automatically in a new Terminal window. IMO it's better to do it manually in a
separate iTerm2 tab instead of having an extra open window.

NOTE: you may think of it as a Rails server but for RN application.

```sh
$ react-native start
```

it's just RN wrapper for `npm start` command.

***UPDATE (2019-06-10)***

it's called Metro (Metro Bundler, `JavaScript bundler for React Native`) now.

### start emulator using specific AVD

```sh
$ emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920
```

### run application in emulator

```sh
$ react-native run-android
```

this command:

- builds application
- doesn't start emulator (it has already been started)
- installs application

the first run might take a while since RN will build the whole Android project.

configuration
-------------

### Dvorak keyboard layout

just enable hardware keyboard (see below).

### Russian keyboard layout

TODO: still not resolved.

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

| emulator (app): `<D-m>` → `Enable Live Reload`

| emulator (app): `<D-m>` → `Enable Hot Reloading`

NOTE: hot reloading is recommended over live reload.

### performance tweaks

| emulator (OS): Applications → `Dev Settings` → `Force GPU rendering` [x]

### enable hardware keyboard

1. <https://stuff.mit.edu/afs/sipb/project/android/docs/tools/devices/managing-avds.html>

edit configuration file _config.ini_ of each AVD in which you want to enable
hardware keyboard (by default AVDs are stored in _~/.android/avd/_).

_~/.android/avd/\<name\>.avd/config.ini_:

```ini
hw.keyboard=yes
```

NOTE: by default `hw.keyboard=no`.

or else it's possible to enable hardware keyboard inside emulator OS:

| emulator (OS): `Settings` → `Language & input` → `Current keyboard` → `Hardware` [x] / `English (US)` [x]

### scroll with 3 fingers

see `Accessibility` section of
[macOS configuration]({% post_url 2016-10-03-macos-configuration %}).

tips
----

### (how to) list all created AVDs

```sh
$ $ANDROID_HOME/tools/emulator -list-avds
Nexus_5X_API_23_x86_64
Nexus_5X_API_28_x86_64
```

### (how to) clean project

1. <https://stackoverflow.com/a/48019133/3632318>

```sh
$ cd android
$ ./gradlew cleanBuildCache
$ ./gradlew clean
```

### (how to) change screen resolution (scale) in emulator

just drag the corner of emulator window.

### (how to) use another Android version in emulator

say, you want to use Android 9 (API level 28):

- install required version of Android SDK Platform packages

  ```sh
  $ sdkmanager 'add-ons;addon-google_apis-google-24'
  $ sdkmanager 'platforms;android-28'
  $ sdkmanager 'system-images;android-28;default;x86_64'
  $ sdkmanager 'system-images;android-28;google_apis;x86_64'
  $ sdkmanager 'build-tools;28.0.3'
  ```

  NOTE: `add-ons;addon-google_apis-google-24` is the latest package available.

- create new AVD using system image of required version

  ```sh
  $ avdmanager create avd \
  --force \
  --package 'system-images;android-28;google_apis;x86_64' \
  --name Nexus_5X_API_28_x86_64 \
  --abi 'google_apis/x86_64' \
  --device 9
  ```

- enable hardware keyboard for new AVD

  ```diff
    ; ~/.android/avd/Nexus_5X_API_28_x86_64.avd/config.ini

  + hw.keyboard=yes
  ```

- start emulator using new AVD

  ```zsh
  # ~/.zshenv

  alias avd6='$ANDROID_HOME/tools/emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920'
  alias avd9='$ANDROID_HOME/tools/emulator -avd Nexus_5X_API_28_x86_64 -gpu host -skin 1080x1920'
  ```

  ```sh
  $ avd9
  ```

### (how to) upload file to emulator

1. <https://stackoverflow.com/questions/5151744/upload-picture-to-emulator-gallery>
2. <https://stackoverflow.com/questions/17928576/refresh-android-mediastore-using-adb>

you can't just drag and drop file onto emulator window just like for iOS
(even though the hint when trying to do so says that the file will be copied
to _/sdcard/Download/_).

perform these steps to upload a file:

- upload file

  ```sh
  $ adb push ~/tmp/image.jpg /sdcard/DCIM/100ANDRO/image.jpg
  ```

- force SD card rescan

  otherwise Gallery doesn't see uploaded file.

  ```sh
  $ adb shell am broadcast -a android.intent.action.MEDIA_MOUNTED -d file:///sdcard
  ```

  also it must be possible to rescan SD card by clicking `SCAN SD CARD` button
  in Applications → `Dev Tools` → `Media Provider`. but it doesn't work -
  something crashes with the message `Unfortunately, Dev Tools has stopped`.

### (how to) delete all application data

this can be useful if, say, it's impossible to log out normally.

```sh
$ adb shell pm list packages
...
package:com.iceperkapp
...
$ adb shell pm clear com.iceperkapp
Success
```

### (how to) build application with 64-bit libraries

> <https://android-developers.googleblog.com/2019/01/get-your-apps-ready-for-64-bit.html>
>
> Starting August 1, 2019:
>
> All new apps and app updates that include native code are required to provide
> 64-bit versions in addition to 32-bit versions when publishing to Google Play.

> <https://developer.android.com/distribute/best-practices/develop/64-bit#building_with_android_studio_or_gradle>
>
> Enabling builds for your native code is as simple as adding the arm64-v8a
> and/or x86_64, depending on the architecture(s) you wish to support, to the
> ndk.abiFilters setting in your app's 'build.gradle' file

```diff
  // android/app/build.gradle

  android {
      defaultConfig {
-         ndk.abiFilters "armeabi-v7a", "x86"
+         ndk.abiFilters "armeabi-v7a", "arm64-v8a", "x86", "x86_64"
      }
```

build application and inspect APK file to ensure 64-bit libraries are present:

> <https://developer.android.com/distribute/best-practices/develop/64-bit#does_your_app_include_64-bit_libraries>
>
> The simplest way to check for 64-bit libraries is to inspect the structure of
> your APK file. When built, the APK will be packaged with any native libraries
> needed by the app.

```sh
$ cd android/app/build/outputs/apk
$ zipinfo -1 app-release.apk | grep \.so$
lib/arm64-v8a/libc++_shared.so
...
lib/armeabi-v7a/libc++_shared.so
...
lib/x86/libc++_shared.so
...
lib/x86_64/libc++_shared.so
...
```

debugging
---------

1. <https://facebook.github.io/react-native/docs/debugging.html>

NOTE: Developer Menu is available only if application is launched via
      `react-native` (that is it's not available if you just start AVD
      and open already installed application by yourself).

### print to device system log

add `console.log('foo');` and reload manually (hot reloading doesn't pick it).

### Developer Menu

| emulator (app): `<D-m>`

### Inspector

| emulator (app): `<D-m>` → `Toggle Inspector`

### Dev Settings

| emulator (OS): Applications → `Dev Settings`

or

| emulator (app): `<D-m>` → `Dev Settings`

### verbose emulator log

```sh
$ avd -verbose
```

### device system log (`log of system messages`)

1. <https://developer.android.com/studio/command-line/logcat.html>

- show messages from `ReactNativeJS` only:

  ```sh
  $ react-native log-android
  ```

- show all system messages (including those from `ReactNativeJS`):

  ```sh
  $ adb logcat
  ```

  this log is very helpful when debugging some system error
  as it shows everything what's happening inside OS.

### reload application in emulator manually

| emulator (app): `<D-m>` → `Reload`

or

| emulator (app): `rr`

### run remote interactive shell

run remote interactive shell:

```sh
$ adb shell
```

run single remote shell command and exit:

```sh
$ adb shell ls
```

### connect to local web server

1. <https://stackoverflow.com/questions/9808560>

- AVD: `10.0.2.2`

  > <https://developer.android.com/studio/run/emulator-networking>
  >
  > If you want to access services running on your development machine loopback
  > interface (a.k.a. 127.0.0.1 on your machine), you should use the special
  > address 10.0.2.2 instead.

  => use `10.0.2.2:3000` instead of `127.0.0.1:3000` to send requests to local
  web server (say, rails server) because emulator runs behind virtual router
  and `10.0.2.2` is a special alias to you host loopback interface (that is,
  `127.0.0.1` on development machine).

- Genymotion: `10.0.3.2`

### show touchable areas

| emulator (app): `<D-m>` → `Toggle Inspector` → `Touchables` (bottom menu)
| `<D-m>` → `Toggle Inspector`

=> touchable areas are still marked with dotted line

### run application in `release` mode

1. <https://facebook.github.io/react-native/docs/signed-apk-android.html#testing-the-release-build-of-your-app>

this can be useful, say, to debug production release crashes:

```sh
$ react-native run-android --variant release
$ react-native log-android
```

NOTE: it's not required to run `react-native start`:

> You can kill any running packager instances, all your framework and JavaScript
> code is bundled in the APK's assets.
