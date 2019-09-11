---
layout: post
title: React Native - Android
date: 2017-05-24 16:27:38 +0300
access: public
comments: true
categories: [react-native, android]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## installation

1. <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>

### install Java 8

see `Could not determine java version from '9.0.4'` error in `troubleshooting`
section below.

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

even though _\$ANDROID_HOME/tools/_ folder contains `emulator` executable, still
it requires `emulator` package to be installed (or else you'll get the very
`Qt library not found` error).

you can launch Android Emulator via any of these executables:

- _\$ANDROID_HOME/tools/emulator_ (I'll use this one for no particular reason)
- _\$ANDROID_HOME/emulator/emulator_

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

### create alias to start emulator using that AVD

1. <https://stackoverflow.com/questions/42718973>
2. <https://developer.android.com/studio/run/emulator-acceleration.html#command-gpu>
   (`-gpu` option)
3. <https://stackoverflow.com/questions/42792947> (`-skin` option)
4. <https://developer.android.com/guide/practices/screens_support.html#testing>
   (available skins)

emulator searches for AVDs in the following directories:

- _\$ANDROID_AVD_HOME_
- _\$ANDROID_SDK_HOME/avd/_
- _~/.android/avd/_

specify `-gpu` and `-skin` options when starting emulator:

```sh
$ emulator -help
$ emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920
```

- `-gpu host` - enables graphics hardware emulation
- `-skin 1080x1920` - changes screen resolution to 1080x1920 (by default a very
  low screen resolution is used)

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

## running

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

**_UPDATE (2019-06-10)_**

it's called Metro (Metro Bundler, `JavaScript bundler for React Native`) now.

### start emulator

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

## configuration

### Dvorak keyboard layout

just enable hardware keyboard (see below).

### Russian keyboard layout

TODO: still not resolved.

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

| emulator                       |
| ------------------------------ |
| `<D-m>` → `Enable Live Reload` |

| emulator                         |
| -------------------------------- |
| `<D-m>` → `Enable Hot Reloading` |

NOTE: hot reloading is recommended over live reload.

### performance tweaks

| emulator (OS)                                             |
| --------------------------------------------------------- |
| Applications → `Dev Settings` → `Force GPU rendering` [x] |

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

| emulator (OS)                                                                              |
| ------------------------------------------------------------------------------------------ |
| `Settings` → `Language & input` → `Current keyboard` → `Hardware` [x] / `English (US)` [x] |

### scroll with 3 fingers

1. [macOS configuration]({% post_url 2016-10-03-macos-configuration %}).

see `Accessibility` section of the linked post.

## tips

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

NOTE: package names might differ not only in major versions (say, `x86_64`
architecture is not available for old versions) - inspect output of
`sdkmanager --list` command before installing new packages.

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

you can't just drag and drop file onto emulator window just like for iOS (even
though the hint when trying to do so says that the file will be copied to
_/sdcard/Download/_).

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

### (how to) increase AVD RAM size

1. <https://stackoverflow.com/questions/9322540>

say, for `Nexus_5X_API_23_x86_64` AVD:

- in AVD config

  > <http://www.androiddocs.com/tools/devices/managing-avds-cmdline.html>
  >
  > Device ram size (hw.ramSize)
  >
  > The amount of physical RAM on the device, in megabytes. Default value is
  > "96".

  ```diff
    # ~/.android/avd/Nexus_5X_API_23_x86_64.avd/config.ini

  + hw.ramSize 768
  ```

- from command line

  use `-memory` option:

  ```sh
  $ emulator -avd Nexus_5X_API_23_x86_64 -memory 768
  ```

## debugging

1. <https://facebook.github.io/react-native/docs/debugging.html>

NOTE: Developer Menu is available only if application is launched via
`react-native` (that is it's not available if you just start AVD and open
already installed application by yourself).

### print to device system log

add `console.log('foo');` and reload manually (hot reloading doesn't pick it).

### Developer Menu

| emulator |
| -------- |
| `<D-m>`  |

### Inspector

| emulator                     |
| ---------------------------- |
| `<D-m>` → `Toggle Inspector` |

### Dev Settings

| emulator (OS)                 |
| ----------------------------- |
| Applications → `Dev Settings` |

or

| emulator                 |
| ------------------------ |
| `<D-m>` → `Dev Settings` |

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

  this log is very helpful when debugging some system error as it shows
  everything what's happening inside OS.

### reload application in emulator manually

| emulator           |
| ------------------ |
| `<D-m>` → `Reload` |

or

| emulator |
| -------- |
| `rr`     |

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
  web server (say, rails server) because emulator runs behind virtual router and
  `10.0.2.2` is a special alias to you host loopback interface (that is,
  `127.0.0.1` on development machine).

- Genymotion: `10.0.3.2`

### show touchable areas

| emulator                                                  |
| --------------------------------------------------------- |
| `<D-m>` → `Toggle Inspector` → `Touchables` (bottom menu) |
| `<D-m>` → `Toggle Inspector`                              |

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

## troubleshooting

see [npm/Yarn - Tips]({% post_url 2017-11-19-npm-yarn-tips %}) for `npm_reset`.

NOTE: all emulator options are omitted for brevity in examples below.

### [0.47.0] method does not override or implement a method from a supertype

```
$ react-native run-android
...
<APP_DIR>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerPackage.java:19: error: method does not override or implement a method from a supertype
  @Override
  ^
Note: <APP_DIR>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerActivityEventListener.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
1 error
:react-native-image-picker:compileReleaseJavaWithJavac FAILED

FAILURE: Build failed with an exception.
```

**solution**

for each package that fails to compile:

- bump its version in _package.json_ (preferably to the latest one)
- `npm install`
- `npm updated <package_name>`

if it doesn't help, try to unlink/link failing package manually:

```sh
$ react-native unlink <package_name>
$ react-native link <package_name>
```

### [0.52.1] Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'

```
$ cd android
$ ./gradlew assembleRelease
...
sentry-cli update to 1.28.1 is available!
run sentry-cli update to update
error: http error: [55] Failed sending data to the peer (SSL_write() returned SYSCALL, errno = 32)
:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* Where:
Script '<APP_DIR>/node_modules/react-native-sentry/sentry.gradle' line: 154

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'.
> Process 'command 'node_modules/sentry-cli-binary/bin/sentry-cli'' finished with non-zero exit value 1
```

**solution**

```sh
$ npm update react-native-sentry
$ npm_reset
```

upgrading `react-native-sentry` didn't help - cleaning cache and reinstalling
all node modules fixed the issue.

### [0.59.9] incompatible types: ReadableArray cannot be converted to ReadableNativeArray

```
$ react-native run-android
...
> Task :react-native-sentry:compileDebugJavaWithJavac FAILED
<APP_DIR>/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java:251: error: incompatible types: Read
ableArray cannot be converted to ReadableNativeArray
            addExceptionInterface(eventBuilder, exception.getString("type"), exception.getString("value"), stacktrace.getArray("frames"));
                                                                                                                              ^
Note: <APP_DIR>/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java uses or overrides a deprecated
 API.
```

**solution**

```sh
$ npm install --save react-native-sentry@0.43.1
```

NOTE: 0.43.1 is the latest version at the time of writing.

### [0.59.9] constructor RNSentryPackage in class RNSentryPackage cannot be applied to given types

```
$ react-native run-android
...
> Task :app:compileDebugJavaWithJavac FAILED
<APP_DIR>/android/app/src/main/java/com/iceperkapp/MainApplication.java:45: error: constructor RNSentryPackage in class RNSentryPac
kage cannot be applied to given types;
            new RNSentryPackage(MainApplication.this),
            ^
  required: no arguments
  found: MainApplication
  reason: actual and formal argument lists differ in length
1 error
```

**solution**

1. <https://github.com/getsentry/react-native-sentry/issues/490#issuecomment-438680937>

```diff
  // android/app/src/main/java/com/iceperkapp/MainApplication.java

- new RNSentryPackage(MainApplication.this),
+ new RNSentryPackage(),
```

### [0.59.9] Cannot fit requested classes in a single dex file

```
$ react-native run-android
...
> Task :app:transformDexArchiveWithExternalLibsDexMergerForDebug FAILED
D8: Cannot fit requested classes in a single dex file (# methods: 66744 > 65536)
```

**solution**

1. <https://developer.android.com/studio/build/multidex>

```diff
  // android/app/build.gradle

  defaultConfig {
      applicationId "com.iceperkapp"
      minSdkVersion rootProject.ext.minSdkVersion
      targetSdkVersion rootProject.ext.targetSdkVersion
+     multiDexEnabled true
      versionCode 97
      versionName "3.17"
      // ...
  }
```

### [0.59.9] The Google Mobile Ads SDK was initialized incorrectly

emulator window:

```
Unfortunately, iceperkapp has stopped
```

```
$ adb logcat
...
06-08 01:19:30.203  3738  3738 E AndroidRuntime: java.lang.RuntimeException: Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider: java.lang.IllegalStateException:
06-08 01:19:30.203  3738  3738 E AndroidRuntime:
06-08 01:19:30.203  3738  3738 E AndroidRuntime: ******************************************************************************
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * The Google Mobile Ads SDK was initialized incorrectly. AdMob publishers    *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * should follow the instructions here: https://goo.gl/fQ2neu to add a valid  *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * App ID inside the AndroidManifest. Google Ad Manager publishers should     *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * follow instructions here: https://goo.gl/h17b6x.                           *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: ******************************************************************************
```

**solution**

1. <https://stackoverflow.com/a/53013453/3632318>
2. <https://developers.google.com/admob/android/quick-start#update_your_androidmanifestxml>

```diff
  <!-- android/app/src/main/AndroidManifest.xml -->

  <manifest>
      <application>
+         <meta-data
+             android:name="com.google.android.gms.ads.APPLICATION_ID"
+             android:value="ca-app-pub-9176480316001296~1177893459"/>
      </application>
  </manifest>
```

### [0.59.9] Invalid application ID

emulator window:

```
Unfortunately, iceperkapp has stopped
```

```
$ adb logcat
...
06-08 10:37:17.754  4378  4378 E AndroidRuntime: java.lang.RuntimeException: Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider: java.lang.IllegalStateException:
06-08 10:37:17.754  4378  4378 E AndroidRuntime:
06-08 10:37:17.754  4378  4378 E AndroidRuntime: ******************************************************************************
06-08 10:37:17.754  4378  4378 E AndroidRuntime: * Invalid application ID. Follow instructions here: https://goo.gl/fQ2neu to *
06-08 10:37:17.754  4378  4378 E AndroidRuntime: * find your app ID.                                                          *
06-08 10:37:17.754  4378  4378 E AndroidRuntime: ******************************************************************************
```

**solution**

make sure to replace slash with tilde in AdMob App ID:

```diff
  <!-- android/app/src/main/AndroidManifest.xml -->

  <manifest>
      <application>
          <meta-data
+             android:name="com.google.android.gms.ads.APPLICATION_ID"
-             android:value="ca-app-pub-9176480316001296/1177893459"/>
+             android:value="ca-app-pub-9176480316001296~1177893459"/>
      </application>
  </manifest>
```

### Could not determine java version from '9.0.4'

the error occurs after updating to JDK 9:

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
Could not determine java version from '9.0.4'.
```

**solution**

1. <https://github.com/facebook/react-native/issues/17688>
2. <https://github.com/facebook/react-native/issues/16536>

probably not the best but the easiest solution is to rollback to JDK 8:

```sh
$ brew cask uninstall java
$ brew tap caskroom/versions
$ brew cask install java8
```

**_UPDATE (2019-07-11)_**

`java8` formula is removed from `homebrew/cask-versions` repository - install
`adoptopenjdk8` formula instead:

```sh
$ brew tap homebrew/cask-versions
$ brew cask install adoptopenjdk8
```

### Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

the error occurs when trying to install application with lower version
(`versionCode` in _android/app/build.gradle_) on device that already has this
application installed with higher version:

```sh
$ react-native run-android
...
:app:installDebug FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: com.android.ddmlib.InstallException: Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
```

**solution**

uninstall application on device (emulator) and build application again.

### No such file or directory - /usr/local/share/android-sdk

1. <https://github.com/caskroom/homebrew-cask/issues/32139>

```sh
$ brew cask uninstall android-sdk
Error: No such file or directory - /usr/local/share/android-sdk
```

**solution**

```sh
$ ln -s /usr/local/Caskroom/android-sdk/25.2.3 /usr/local/share/android-sdk
$ brew cask uninstall android-sdk
```

### repositories.cfg could not be loaded

1. <https://askubuntu.com/questions/885658>

```sh
$ sdkmanager --list
Warning: File /Users/tap/.android/repositories.cfg could not be loaded.
```

**solution**

```sh
$ touch ~/.android/repositories.cfg
```

### truncated package paths in output from sdkmanager

1. <https://stackoverflow.com/questions/42460205>

```sh
$ sdkmanager --list
...
Available Packages:
  Path                              | Version      | Description
  -------                           | -------      | -------
  add-ons;addon-g..._apis-google-15 | 3            | Google APIs
  add-ons;addon-g..._apis-google-16 | 4            | Google APIs
```

**solution**

```sh
$ sdkmanager --list --verbose
```

**_UPDATE (2019-06-07)_**

`sdkmanager` doesn't truncate package paths anymore.

### Qt library not found

1. <https://stackoverflow.com/questions/40931254>

```
$ emulator -avd Nexus_5X_API_23_x86_64
[140735129182208]:ERROR:./android/qt/qt_setup.cpp:28:Qt library not found at ../emulator/lib64/qt/lib
Could not launch '../emulator/qemu/darwin-x86_64/qemu-system-x86_64': No such file or directory
```

**solution**

it's necessary to cd to emulator directory and use relative path:

```sh
$ cd /usr/local/share/android-sdk/emulator
$ ./emulator -avd Nexus_5X_API_23_x86_64
```

or else create an alias in _~/.zshenv_:

```conf
export ANDROID_HOME=/usr/local/share/android-sdk

alias emulator='cd $ANDROID_HOME/emulator && ./emulator'
alias avd='emulator -avd Nexus_5X_API_23_x86_64'
```

**_UPDATE_**

after updating emulator to version 26.1.4.0 it's no longer necessary to cd to
emulator directory but still you cannot rely on `PATH` - use absolute path:

_~/.zshenv_:

```conf
alias avd='$ANDROID_HOME/tools/emulator -avd Nexus_5X_API_23_x86_64'
# `Qt library not found` again:
#alias avd='emulator -avd Nexus_5X_API_23_x86_64'
```

### Cannot find AVD system path

NOTE: the error occurs sporadically and might disappear on its own.

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT
```

**solution**

_~/.zshenv_:

```conf
export ANDROID_SDK_ROOT=/usr/local/share/android-sdk
```

### The SDK directory does not exist

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> The SDK directory '/Users/tap/Library/Android/sdk' does not exist.
```

**solution**

if you previously used Android Studio to run application it might have created
_android/local.properties_ file in project with the following content:

```conf
sdk.dir=/Users/tap/Library/Android/sdk
```

that is RN now expects Android SDK to be installed in _~/Library/Android/sdk/_
(this is where Android Studio would install it).

there are 2 ways to solve the problem:

- create symlink that points to actual Android SDK root:

  ```sh
  $ mkdir ~/Library/Android
  $ ln -s $ANDROID_HOME ~/Library/Android/sdk
  ```

  or edit _android/local.properties_ directly:

  ```
  # export ANDROID_HOME=/usr/local/share/android-sdk
  sdk.dir=/usr/local/share/android-sdk
  ```

- just comment out or remove that line

  RN will search for Android SDK in `$ANDROID_HOME` then.

### Could not get unknown property 'MYAPP_RELEASE_STORE_FILE'

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* Where:
Build file '<APP_DIR>/android/app/build.gradle' line: 108

* What went wrong:
A problem occurred evaluating project ':app'.
> Could not get unknown property 'MYAPP_RELEASE_STORE_FILE' for SigningConfig_Decorated...
```

**solution**

1. <https://facebook.github.io/react-native/docs/signed-apk-android.html>

create dummy _~/.gradle/gradle.properties_ file:

```conf
MYAPP_RELEASE_STORE_FILE=test
MYAPP_RELEASE_KEY_ALIAS=test
MYAPP_RELEASE_STORE_PASSWORD=test
MYAPP_RELEASE_KEY_PASSWORD=test
```

since these variables (`MYAPP_RELEASE_STORE_FILE`, etc.) are mentioned in
_android/app/build.gradle_ project file and build fails if they were not set.

### DeviceException: No connected devices!

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!
```

**solution**

start emulator before running application.

### Could not find com.facebook.react:react-native

1. <https://github.com/oblador/react-native-vector-icons/issues/480>
2. <https://github.com/facebook/react-native/issues/14223#issuecomment-304447493>

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > A problem occurred configuring project ':react-native-push-notification'.
      > Could not resolve all dependencies for configuration ':react-native-push-notification:_debugPublishCopy'.
         > Could not find com.facebook.react:react-native:0.42.0.
           Required by:
               myapp:react-native-push-notification:unspecified
```

**solution**

add this snippet to _android/build.gradle_ and specify actual RN version in it
(make sure to update this value every time RN version changes):

```gradle
allprojects {
  configurations.all {
    resolutionStrategy {
      eachDependency { DependencyResolveDetails details ->
        if (details.requested.group == 'com.facebook.react' && details.requested.name == 'react-native') {
            details.useVersion '0.42.3' # specify actual RN version here
        }
      }
    }
  }
}
```

### CANNOT TRANSLATE guest DNS ip

1. <https://github.com/facebook/react-native/issues/13340>

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
...
CANNOT TRANSLATE guest DNS ip
```

**solution**

issue is resolved after updating emulator to version 26.1.4.0:

```sh
$ sdkmanager --update
```

### no application launcher icon on home screen in emulator

1. <https://developer.android.com/guide/practices/ui_guidelines/icon_design_launcher.html>

application doesn't have launcher icon on home screen in emulator.

**solution**

`react-native run-android` installs application but doesn't add application
launcher icon on home screen - you can do it manually:

- go to applications
- click and hold application icon
- drag application icon to home screen

### Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

```
$ react-native run-android
...
com.android.ddmlib.InstallException: Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
```

**solution**

1. <https://stackoverflow.com/questions/13808599>

RN fails to install application with lower versionCode than currently installed
in emulator - uninstall application with higher versionCode inside emulator and
build again.

### You have not accepted the license agreements of the following SDK components

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > A problem occurred configuring project ':react-native-svg'.
      > You have not accepted the license agreements of the following SDK components:
        [Android SDK Build-Tools 25.0.1, Android SDK Platform 25].
        Before building your project, you need to accept the license agreements and complete the installation of the missing components using the Android Studio SDK Manager.
        Alternatively, to learn how to transfer the license agreements from one workstation to another, go to http://d.android.com/r/studio-ui/export-licenses.html
```

**solution**

```sh
$ sudo sdkmanager --licenses
/ accept all licenses
```

### Configuration with name 'default' not found.

```
$ react-native run-android
...
Building and installing the app on the device (cd android && ./gradlew installDebug)...

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > Configuration with name 'default' not found.
```

**solution**

I haven't installed new dependencies after rebase:

```
$ npm install
```

### Manifest merger failed

```
$ react-native run-android
...
> Task :app:processDebugManifest FAILED
<APP_DIR>/android/app/src/debug/AndroidManifest.xml:22:18-91 Error:
        Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91
        is also present at [androidx.core:core:1.0.0] AndroidManifest.xml:22:18-86 value=(androidx.core.app.CoreComponentFactory).
        Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:7:6-11:8 to override.

See http://g.co/androidstudio/manifest-merger for more information about the manifest merger.


FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:processDebugManifest'.
> Manifest merger failed : Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91
        is also present at [androidx.core:core:1.0.0] AndroidManifest.xml:22:18-86 value=(androidx.core.app.CoreComponentFactory).
        Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:7:6-11:8 to override.
```

**solution**

[1]: {% post_url 2019-09-06-react-native-060 %}

error is caused by AndroidX libraries leaking into application dependencies -
unless you upgrade RN to 0.60+, it's easier to force use of Android Support
Libraries - see [2019-09-06-react-native-060][1] for details.

### Could not get unknown property 'android' for root project 'iceperkapp' of type org.gradle.api.Project

1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-adding-the-gradle-plugin>

```
$ cd android
$ ./gradlew assembleRelease
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'iceperkapp'.
> Could not get unknown property 'android' for root project 'iceperkapp' of type org.gradle.api.Project.
```

the error occurred right after configuring `react-native-onesignal` package by
following official `React Native SDK Setup` guide (see the link).

**solution**

official guide contains error:

> <https://github.com/geektimecoil/react-native-onesignal/issues/550#issuecomment-396015932>
>
> you applied the onesignal-gradle-plugin to your root build.gradle or
> android/build.gradle instead of the one in app/build.gradle

```diff
  // android/build.gradle

- apply plugin: 'com.onesignal.androidsdk.onesignal-gradle-plugin'
```

```diff
  // android/app/build.gradle

+ apply plugin: 'com.onesignal.androidsdk.onesignal-gradle-plugin'
```

### ENOTEMPTY: directory not empty

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

### FATAL EXCEPTION IN SYSTEM PROCESS: WindowManager

1. <https://stackoverflow.com/questions/26808669/android-avd-is-crashing-with-nullpointerexception-on-startup>

error occurs when starting emulator using AVD with Android 4.4 => emulator is
stuck on splash screen:

```
$ adb logcat
...
E/AndroidRuntime(21204): *** FATAL EXCEPTION IN SYSTEM PROCESS: WindowManager
E/AndroidRuntime(21204): java.lang.NullPointerException
E/AndroidRuntime(21204): 	at com.android.server.display.LocalDisplayAdapter$LocalDisplayDevice.getDisplayDeviceInfoLocked(LocalDisplayAdapter.java:147)
E/AndroidRuntime(21204): 	at com.android.server.display.DisplayManagerService.handleDisplayDeviceAddedLocked(DisplayManagerService.java:852)
```

**solution**

TODO: still not resolved.

### Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider

application crashes on startup:

```
$ adb logcat
...
E/AndroidRuntime( 2579): FATAL EXCEPTION: main
E/AndroidRuntime( 2579): Process: com.iceperkapp, PID: 2579
E/AndroidRuntime( 2579): java.lang.RuntimeException: Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider: java.lang.ClassNotFoundException: Didn't find class "com.google.android.gms.ads.MobileAdsInitProvider" on path: DexPathList[[zip file "/data/app/com.iceperkapp-2.apk"],nativeLibraryDirectories=[/data/app-lib/com.iceperkapp-2, /vendor/lib, /system/lib]]
E/AndroidRuntime( 2579):   at android.app.ActivityThread.installProvider(ActivityThread.java:4793)
...
E/AndroidRuntime( 2579): Caused by: java.lang.ClassNotFoundException: Didn't find class "com.google.android.gms.ads.MobileAdsInitProvider" on path: DexPathList[[zip file "/data/app/com.iceperkapp-2.apk"],nativeLibraryDirectories=[/data/app-lib/com.iceperkapp-2, /vendor/lib, /system/lib]]
E/AndroidRuntime( 2579):   at dalvik.system.BaseDexClassLoader.findClass(BaseDexClassLoader.java:56)
...
W/ActivityManager( 1564):   Force finishing activity com.iceperkapp/.MainActivity
```

**solution**

1. <https://stackoverflow.com/a/54501907/3632318>

multidex support wasn't properly configured:

```diff
  // android/app/build.gradle

  android {
      defaultConfig {
          // ...
+         multiDexEnabled true
      }
  }

  dependencies {
      // ...
+     implementation 'com.android.support:multidex:1.0.3'
  }
```

```diff
  // android/app/src/main/java/com/iceperkapp/MainApplication.java

- import android.app.Application;
+ import android.support.multidex.MultiDexApplication;

  // ...

- public class MainApplication extends Application implements ReactApplication {
+ public class MainApplication extends MultiDexApplication implements ReactApplication {
```

### [0.60.5] Keystore file '<APP_DIR>/android/app/debug.keystore' not found for signing config 'debug'

```
$ react-native run-android
...
* What went wrong:
Execution failed for task ':app:validateSigningDebug'.
> Keystore file '<APP_DIR>/android/app/debug.keystore' not found for signing config 'debug'.
```

**solution**

1. <https://github.com/facebook/react-native/issues/25629#issuecomment-511209583>

```sh
$ cd android/app
$ keytool -genkey -v -keystore debug.keystore -storepass android \
  -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 \
  -validity 10000
```

or else it might be possible to use production keystore in debug mode:

```sh
$ cd android/app
$ cp <MY_APP>.keystore debug.keystore
```

### [0.60.5] Program type already present: com.sbugert.rnadmob.BuildConfig

```
$ react-native run-android
...
* What went wrong:
Execution failed for task ':app:transformClassesWithMultidexlistForDebug'.
> com.android.build.api.transform.TransformException: Error while generating the main dex list:
  Error while merging dex archives:
  Program type already present: com.sbugert.rnadmob.BuildConfig
  Learn how to resolve the issue at https://developer.android.com/studio/build/dependencies#duplicate_classes.
```

**solution**

> <https://developer.android.com/studio/build/dependencies#duplicate_classes>
>
> If a class appears more than once on the runtime classpath, you get an error
> similar to the following:
>
> Program type already present com.example.MyClass

in my case `react-native-admob` (or `RNAdMob`) was referenced twice in:

- _android/app/build.gradle_ (`implementation`)
- _android/settings.gradle_ (`include`)
- _android/app/src/main/java/com/iceperkapp/MainApplication.java_ (`import`)

solution is to remove it completely everywhere (autolinking should handle it
automatically).

### [0.60.5] com.android.ddmlib.InstallException: INSTALL_FAILED_UPDATE_INCOMPATIBLE

```
$ react-native run-android
...
* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: com.android.ddmlib.InstallException: INSTALL_FAILED_UPDATE_INCOMPATIBLE
```

**solution**

I generated debug keystore before - it's necessary to reinstall application:

> <https://github.com/meteor/meteor/issues/5266#issuecomment-142789166>
>
> INSTALL_FAILED_UPDATE_INCOMPATIBLE is an Android error that means an apk
> cannot be installed because its signature is incompatible with the currently
> installed version. This happens when you try to update a signed release build
> with a debug build for instance. The solution is to uninstall the existing app
> from the device.


### [0.60.5] Could not find androidx.multidex:multidex:$multidex_version.

```
$ alias build_apk='cd android && ./gradlew assembleRelease; cd ..'
$ build_apk
...
> Task :app:lintVitalRelease FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:lintVitalRelease'.
> Could not resolve all artifacts for configuration ':app:debugUnitTestRuntimeClasspath'.
   > Could not find androidx.multidex:multidex:$multidex_version.
     Required by:
         project :app
```

**solution**

even though it's officially recommended to use `multidex_version` variable:

> <https://developer.android.com/studio/build/multidex#mdex-pre-l>
>
> ```groovy
> dependencies {
>     def multidex_version = "2.0.1"
>     implementation 'androidx.multidex:multidex:$multidex_version'
> }
> ```

this variable is not interpolated for some reason when `:app:lintVitalRelease`
task is run => hard-code multidex version:

```diff
  // android/app/build.gradle

- def multidex_version = "2.0.1"
- implementation 'androidx.multidex:multidex:$multidex_version'
+ implementation 'androidx.multidex:multidex:2.0.1'
```

### [0.60.5] Could not find androidx.multidex:multidex:$multidex_version.

```
$ alias build_apk='cd android && ./gradlew assembleRelease; cd ..'
$ build_apk
...
> Task :app:lintVitalRelease FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:lintVitalRelease'.
> Could not resolve all artifacts for configuration ':react-native-image-picker:debugUnitTestRuntimeClasspath'.
   > Could not resolve org.easytesting:fest-assert-core:2.0M10.
     Required by:
         project :react-native-image-picker
      > Could not resolve org.easytesting:fest-assert-core:2.0M10.
         > Could not get resource 'https://jitpack.io/org/easytesting/fest-assert-core/2.0M10/fest-assert-core-2.0M10.pom'.
            > Could not GET 'https://jitpack.io/org/easytesting/fest-assert-core/2.0M10/fest-assert-core-2.0M10.pom'. Received status code 522 from server: Origin Connection Time-out
```

**solution**

RN Upgrade Helper suggests adding JitPack repository but since it cannot be
resolved the easiest solution is to remove it altogether:

```diff
  // android/build.gradle

  allprojects {
      repositories {
-         maven { url "https://jitpack.io" }
          // ...
      }
  }
```
