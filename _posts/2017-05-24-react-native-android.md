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

1. <https://facebook.github.io/react-native/docs/getting-started.html>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>
3. <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>

> <https://facebook.github.io/react-native/docs/getting-started.html>
>
> Node, Watchman, JDK
>
> The React Native CLI

```sh
$ brew install node watchman
$ brew cask install java8
$ npm install -g react-native-cli
```

### install Android SDK

```sh
$ brew cask install java
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
  $ sdkmanager 'add-ons;addon-google_apis-google-28'
  $ sdkmanager 'platforms;android-28'
  $ sdkmanager 'system-images;android-28;default;x86_64'
  $ sdkmanager 'system-images;android-28;google_apis;x86_64'
  $ sdkmanager 'build-tools;28.0.3'
  ```

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

troubleshooting
---------------

NOTE: all emulator options are omitted for brevity in examples below.

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

***UPDATE (2019-06-07)***

`sdkmanager` doesn't truncate package paths anymore.

### Qt library not found

1. <https://stackoverflow.com/questions/40931254>

```sh
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

***UPDATE***

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

if you previously used Android Studio to run application it might have
created _android/local.properties_ file in project with the following content:

```conf
sdk.dir=/Users/tap/Library/Android/sdk
```

that is RN now expects Android SDK to be installed in
_~/Library/Android/sdk/_ (this is where Android Studio would install it).

there are 2 ways to solve the problem:

- create symlink that points to actual Android SDK root:

  ```sh
  $ mkdir ~/Library/Android
  $ ln -s $ANDROID_HOME ~/Library/Android/sdk
  ````

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

issue is resolved after updating emulator to version 26.1.4.0
via `sdkmanager --update`.

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

### method does not override or implement a method from a supertype

the error occurred after upgrading RN to 0.47.0.

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

> <https://medium.com/@yathousen/the-day-google-decided-to-shake-the-react-native-community-4ba5cdd33388>
>
> It was June 17th, 2019, Android released their new package names for their
> DEPRECATED support libraries + other tools inside AndroidX...
>
> Basically, all of the basic libraries that Google provide for the Android +
> React Native Community, on the last or close to last versions were affected.
> In some cases, it was the fault of the react-native libraries maintainers
> for using imports like + symbols that are ambiguous and generally take the
> last version available of the library and in some other cases was caused by
> the fact that the developer was actually using new features from the latest
> release.

> <https://stackoverflow.com/a/52517772/3632318>
>
> AndroidX is redesigned library to make package names more clear.

> <https://stackoverflow.com/a/55849025/3632318>
>
> AndroidX is a major improvement to the original Android Support Library.
>
> AndroidX fully replaces the Support Library by providing feature parity and
> new libraries.

> <https://stackoverflow.com/a/54533702/3632318>
>
> One app should use either AndroidX or old Android Support libraries. That's
> why you faced this issue.
>
> So the solution is to use either AndroidX or old Support Library.

so the problem is that application must be using both new AndroidX and old
Android Support libraries - hence the error.

it's possible to examine all application dependencies using this command:

```sh
$ cd android
$ ./gradlew app:dependencies
```

AFAIU there can be either `com.android.support` or `androidx.*` libraries in
the output but not both.

the reason why AndroidX libraries leak into current application dependencies
is that many RN libraries use the latest versions of Google libraries instead
of specific ones, say (note `+` symbol):

```groovy
// https://github.com/zo0r/react-native-push-notification/blob/c4c961cf3a76bffe4e9e36cf00201611eecbb898/android/build.gradle

dependencies {
    // ...
    implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
    // ...
    implementation "com.google.firebase:firebase-messaging:${safeExtGet('firebaseVersion', '+')}"
}
```

but the latest versions of Google started to depend on AndroidX libraries since
June 17th, 2019 and it was a breaking change.

there might be several solutions to this problem:

#### [RECOMMENDED] try upgrading RN packages

1. <https://github.com/react-native-community/async-storage/issues/128#issuecomment-505423631>

maybe maintainers of RN packages have fixed their _build.gradle_ files to use
specific versions of Google libraries:

```sh
$ yarn upgrade --pattern react-native
```

it was the case with `react-native-device-info` package:

> <https://github.com/react-native-community/react-native-device-info/pull/693>
>
> With the latest firebase/gcm release, the latest version (+) now brings in
> androidx dependencies. react-native won't be ready for androidx until 0.60
> so constrain the gcm version to 16.1.0 (the latest pre-androidx version).

```diff
  # https://github.com/react-native-community/react-native-device-info/pull/693/files#diff-7ae5a9093507568eabbf35c3b0665732

  dependencies {
    implementation "com.facebook.react:react-native:${safeExtGet('reactNativeVersion', '+')}"
-   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
+   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '16.1.0')}"
```

#### fork RN packages

1. <https://github.com/react-native-community/react-native-device-info/pull/693>

if RN package is not actively maintained it might take a long time till it's
fixed, so it makes sense to fork and fix it by yourself (just like it's done
in the linked PR which is also mentioned previously).

NOTE: you can open a PR to the upstream repository from a fork.

#### force use of specific versions of Google libraries

1. <https://github.com/facebook/react-native/issues/25292#issuecomment-502998885>
2. <https://medium.com/@suchydan/how-to-solve-google-play-services-version-collision-in-gradle-dependencies-ef086ae5c75f>
3. <https://github.com/facebook/react-native/issues/25307#issuecomment-503516458>
4. <https://medium.com/mindorks/avoiding-conflicts-in-android-gradle-dependencies-28e4200ca235>

in this solution it's not necessary to modify Google libraries but your own
_android/app/build.gradle_ file only.

perform these steps for each RN package which depends on AndroidX libraries
until there are no AndroidX dependencies (run `gradlew app:dependencies` to
find such RN packages as described above):

- find Google libraries in RN package dependencies

  libraries inside `com.google.android.gms` and `com.google.firebase` groups
  are affected:

  ```groovy
  // https://github.com/zo0r/react-native-push-notification/blob/c4c961cf3a76bffe4e9e36cf00201611eecbb898/android/build.gradle

  dependencies {
      // ...
      implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
      // ...
      implementation "com.google.firebase:firebase-messaging:${safeExtGet('firebaseVersion', '+')}"
  }
  ```

- exclude these libraries from RN package dependencies in your application

  ```diff
    // android/app/build.gradle

    dependencies {
  -     implementation project(':react-native-push-notification')
  +     implementation(project(':react-native-push-notification')) {
  +       exclude group: 'com.google.android.gms', module: 'play-services-gcm'
  +       exclude group: 'com.google.firebase', module: 'firebase-messaging'
  +     }
  ```

- use specific versions of these libraries in your application

  ```diff
    // android/app/build.gradle

    dependencies {
        // ...
  +     implementation("com.google.android.gms:play-services-base:16.1.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-basement:16.2.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-gcm:16.1.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-stats:16.0.1") {
  +       force = true;
  +     }
  +     implementation("com.google.firebase:firebase-messaging:18.0.0") {
  +       force = true;
  +     }
  +     implementation("com.google.firebase:firebase-iid:18.0.0") {
  +       force = true;
  +     }
    }
  ```

  a rule of thumb is not to use June versions, the last but one versions will
  do in most cases.

  see [Maven repository](https://mvnrepository.com/) for the list of library
  versions and corresponding release dates.

  it might be necessary to force versions of both direct and transitive RN
  package dependencies (just like in example above) - inspect the output of
  `gradlew app:dependencies` command after adding all direct dependencies.
