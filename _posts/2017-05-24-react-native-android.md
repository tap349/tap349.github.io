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

## installation

1. <https://facebook.github.io/react-native/docs/getting-started.html>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>
3. <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>

install prerequisites and react-native-cli:

```sh
$ brew install node watchman
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
# `adb` (Android Debug Bridge)
path=($path $ANDROID_HOME/platform-tools)
```

### install Android SDK Platform packages (version 23)

<https://developer.android.com/studio/command-line/sdkmanager.html>

- list all packages

  list all packages along with outdated packages
  (installed packages that have new version available).

  ```sh
  $ sdkmanager --list
  ```

- install required packages

  install or update required packages if they are already installed:

  > Google APIs

  > Android SDK Platform 23 (Android 6.0 (Marshmallow))

  > Intel x86 Atom_64 System Image

  > Google APIs Intel x86 Atom_64 System Image

  ```sh
  $ sdkmanager 'add-ons;addon-google_apis-google-23'
  $ sdkmanager 'platforms;android-23'
  $ sdkmanager 'system-images;android-23;default;x86_64'
  $ sdkmanager 'system-images;android-23;google_apis;x86_64'
  ```

  > Android SDK Build-Tools 23.0.1

  ```sh
  $ sdkmanager 'build-tools;23.0.1'
  ```

- update installed packages

  update all installed packages to their latest versions:

  ```sh
  $ sdkmanager --update
  ```

  this is preferred way to update Android SDK packages since
  `android-sdk` brew formula is not updated frequently.

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
- _$HOME/.android/avd/_

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

according to emulator's log there is no need to install HAXM separately
(it appears to have been already enabled):

```sh
Hax is enabled
Hax ram_size 0x40000000
HAX is working and emulator runs in fast virt mode.
```

## running

NOTE: emulator for Android is called Android Emulator
      (will be referred to as just emulator for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS server) (optional)

if you don't start it manually it will be started automatically by
`react-native run-android` in a new Terminal window. IMO it's better to do
it manually in a separate iTerm2 tab instead of having an extra open window.

NOTE: you may think of it as a Rails server but for RN application.

```sh
$ npm start
```

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

## configuration

### Russian keyboard layout

TODO: still not resolved.

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

NOTE: hot reloading is recommended over live reload.

- `<D-m>` -> `Enable Live Reload`
- `<D-m>` -> `Enable Hot Reloading`

### performance tweaks

- Android:

  `<D-m>` -> `Dev Settings` -> `Force GPU rendering`

- emulator:

  Settings -> `OpenGL ES renderer (requires restart)` -> `SwiftShader`

### enable hardware keyboard

1. <https://stuff.mit.edu/afs/sipb/project/android/docs/tools/devices/managing-avds.html>

edit configuration file _config.ini_ of each AVD in which you want to enable
hardware keyboard (by default AVDs are stored in _$HOME/.android/avd/_).

_$HOME/.android/avd/\<name\>.avd/config.ini_:

```ini
hw.keyboard=yes
```

by default `hw.keyboard=no`.

### scroll with 3 fingers

see `Accessibility` section of
[macOS configuration]({% post_url 2016-10-03-macos-configuration %}).

## tips

### change screen resolution (scale) in emulator

just drag the corner of emulator window.

### use another Android version in emulator

- install required version of Android SDK Platform packages
  (version 23 = Android 6)
- create new AVD using system image of required version
- start emulator using new AVD

### upload file to emulator

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
  in Applications -> `Dev Tools` -> `Media Provider`. but it doesn't work -
  something crashes with the message `Unfortunately, Dev Tools has stopped`.

### delete all application data

this can be useful if, say, it's impossible to log out normally.

```sh
$ adb shell pm list packages
...
package:com.iceperkapp
...
$ adb shell pm clear com.iceperkapp
Success
```

## debugging

1. <https://facebook.github.io/react-native/docs/debugging.html>

NOTE: Developer Menu is available only if application is launched
      via `react-native` (that is it's not available if you just start
      AVD and open already installed application by yourself).

### print to device system log

add `console.log('foo');` and reload manually (hot reloading doesn't pick it).

### Developer Menu

`<D-m>`

### Inspector

- `<D-m>` -> `Toggle Inspector`

### Dev Settings

in Android:

- Applications -> `Dev Settings`
- `<D-m>` -> `Dev Settings`

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
  as it shows everything that's happening inside OS.

### reload application in emulator manually

- `rr`
- `<D-m>` -> `Reload`

### run remote interactive shell

run remote interactive shell:

```sh
$ adb shell
```

run single remote shell command:

```sh
$ adb shell ls
```

### connect to local web server

1. <https://stackoverflow.com/questions/9808560>

use `10.0.2.2:3000` instead of `127.0.0.1:3000` to send requests to local web
server (say, puma) because emulator runs behind virtual router and `10.0.2.2`
is a special alias to you host loopback interface (that is, `127.0.0.1` on
development machine).

### show touchable areas

- `<D-m>` -> `Toggle Inspector`
- click `Touchables` in bottom menu
- `<D-m>` -> `Toggle Inspector`
- touchable areas are still marked with dotted line

## troubleshooting

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

### Qt library not found

1. <https://stackoverflow.com/questions/40931254>

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
[140735129182208]:ERROR:./android/qt/qt_setup.cpp:28:Qt library not found at ../emulator/lib64/qt/lib
Could not launch '../emulator/qemu/darwin-x86_64/qemu-system-x86_64': No such file or directory
```

**solution**

```sh
$ cd /usr/local/share/android-sdk/emulator
$ ./emulator -avd Nexus_5X_API_23_x86_64
```

or else create an alias in _~/.zshenv_:

```conf
export ANDROID_HOME=/usr/local/share/android-sdk

alias avd='emulator -avd Nexus_5X_API_23_x86_64'
alias emulator='cd $ANDROID_HOME/emulator && ./emulator'
```

**UPDATE**

after updating emulator to version 26.1.4.0 it's no longer necessary to
cd to emulator directory.

_~/.zshenv_:

```conf
alias avd='$ANDROID_HOME/emulator/emulator -avd Nexus_5X_API_23_x86_64'
```

we cannot use just `emulator` since it points to `/usr/local/bin/emulator`
instead of `$ANDROID_HOME/emulator/emulator`.

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

```sh
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

- just comment out or remove that line

  RN will search for Androd SDK in `$ANDROID_HOME` then.

### Could not get unknown property 'MYAPP_RELEASE_STORE_FILE'

```sh
$ react-native run-android
...
FAILURE: Build failed with an exception.

* Where:
Build file '<app_dir>/android/app/build.gradle' line: 108

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

```sh
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

```sh
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

`react-native run-android` installs application but doesn't add
application launcher icon on home screen - you can do it manually:

- go to applications
- click and hold application icon
- drag application icon to home screen

### Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

```sh
$ react-native run-android
...
com.android.ddmlib.InstallException: Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
```

**solution**

1. <https://stackoverflow.com/questions/13808599>

RN fails to install application with lower versionCode than currently installed
in emulator - uninstall application with higher versionCode and build again.

### You have not accepted the license agreements of the following SDK components

```sh
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

error occurs after upgrading RN to 0.47.0.

```sh
$ react-native run-android
...
<app_dir>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerPackage.java:19: error: method does not override or implement a method from a supertype
  @Override
  ^
Note: <app_dir>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerActivityEventListener.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
1 error
:react-native-image-picker:compileReleaseJavaWithJavac FAILED

FAILURE: Build failed with an exception.
```

**solution**

for each package that fails to compile:

- bump its version in _package.json_ (preferably to the latest one)
- `yarn`
- `yarn upgrade <package_name>`
