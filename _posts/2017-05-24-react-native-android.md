---
layout: post
title: React Native - Android
date: 2017-05-24 16:27:38 +0300
access: public
categories: [react-native, android]
---

<!-- more -->

* TOC
{:toc}

## installation

- <https://facebook.github.io/react-native/docs/getting-started.html>
- <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>
- <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>

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
# `adb`
path=($path $ANDROID_HOME/platform-tools)
```

### install Android SDK Platform packages

<https://developer.android.com/studio/command-line/sdkmanager.html>

list all available packages:

```sh
$ sdkmanager --list
```

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

### create new AVD (Android Virtual Device)

<https://developer.android.com/studio/command-line/avdmanager.html>

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

### start AVD

- <https://stackoverflow.com/questions/42718973>
- <https://developer.android.com/studio/run/emulator-acceleration.html#command-gpu>
  (`-gpu` option)
- <https://stackoverflow.com/questions/42792947>
  (`-skin` option)
- <https://developer.android.com/guide/practices/screens_support.html#testing>
  (available skins)

emulator searches for AVDs in the following directories:

- `$ANDROID_AVD_HOME`
- `$ANDROID_SDK_HOME/avd`
- `$HOME/.android/avd`

```sh
$ emulator -help
$ emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920
```

make sure to specify both `-gpu` and `-skin` options:

- `-gpu host` - enables graphics hardware emulation
- `-skin 1080x1920` - changes screen resolution to WXGA
  (by default a very low screen resolution is used)

some skin resolutions have corresponding skin names (see link above).

according to emulator's log there is no need to install HAXM separately:

```sh
Hax is enabled
Hax ram_size 0x40000000
HAX is working and emulator runs in fast virt mode.
```

## running

### start server

```sh
$ rails server
```

### start AVD

```sh
$ emulator -avd Nexus_5X_API_23_x86_64 -gpu host -skin 1080x1920
```

### run application

```sh
$ react-native run-android
```

the first run might take a long time since RN will try to
download and install all required Android libraries.

## troubleshooting

### brew cannot uninstall android-sdk

<https://github.com/caskroom/homebrew-cask/issues/32139>

```sh
$ brew cask uninstall android-sdk
Error: No such file or directory - /usr/local/share/android-sdk
```

solution:

```sh
$ ln -s /usr/local/Caskroom/android-sdk/25.2.3 /usr/local/share/android-sdk
$ brew cask uninstall android-sdk
```

### warning when running tools from android-sdk

<https://askubuntu.com/questions/885658>

```sh
$ sdkmanager --list
Warning: File /Users/tap/.android/repositories.cfg could not be loaded.
```

solution:

```sh
$ touch ~/.android/repositories.cfg
```

### truncated package paths in output from sdkmanager

<https://stackoverflow.com/questions/42460205>

```sh
$ sdkmanager --list
...
Available Packages:
  Path                              | Version      | Description
  -------                           | -------      | -------
  add-ons;addon-g..._apis-google-15 | 3            | Google APIs
  add-ons;addon-g..._apis-google-16 | 4            | Google APIs
```

solution:

```sh
$ sdkmanager --list --verbose
```

### emulator cannot start AVD

<https://stackoverflow.com/questions/40931254/could-not-launch-emulator-in-android-studio>

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
[140735129182208]:ERROR:./android/qt/qt_setup.cpp:28:Qt library not found at ../emulator/lib64/qt/lib
Could not launch '../emulator/qemu/darwin-x86_64/qemu-system-x86_64': No such file or directory
```

solution:

```sh
$ cd /usr/local/share/android-sdk/emulator
$ ./emulator -avd Nexus_5X_API_23_x86_64
```

or else create alias in _~/.zshenv_:

```conf
alias emulator='cd /usr/local/share/android-sdk/emulator && ./emulator'
```

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
```

### emulator cannot find AVD

NOTE: the error occurs sporadically and might disappear on its own.

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT
```

solution:

_~/.zshenv_:

```conf
export ANDROID_SDK_ROOT=/usr/local/share/android-sdk
```

### application build fails (Android SDK directory doesn't exist)

```sh
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> The SDK directory '/Users/tap/Library/Android/sdk' does not exist.
```

solution:

if you previously used Android Studio to run application it might have
created _android/local.properties_ project file with the following content:

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

  RN will search for Androd SDK in `ANDROID_HOME` then.

### application build fails (unknown property 'MYAPP_RELEASE_STORE_FILE')

```sh
FAILURE: Build failed with an exception.

* Where:
Build file '/Users/tap/dev/complead/iceperkapp/android/app/build.gradle' line: 108

* What went wrong:
A problem occurred evaluating project ':app'.
> Could not get unknown property 'MYAPP_RELEASE_STORE_FILE' for SigningConfig_Decorated...
```

solution:

<https://facebook.github.io/react-native/docs/signed-apk-android.html>

create dummy _~/.gradle/gradle.properties_ file:

```conf
MYAPP_RELEASE_STORE_FILE=test
MYAPP_RELEASE_KEY_ALIAS=test
MYAPP_RELEASE_STORE_PASSWORD=test
MYAPP_RELEASE_KEY_PASSWORD=test
```

since these variables (`MYAPP_RELEASE_STORE_FILE`, etc.) are mentioned in
_android/app/build.gradle_ project file and build fails if they were not set.

### application build fails (no connected devices)

```sh
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!
```

solution:

start AVD before running application.
