---
layout: post
title: React Native - Troubleshooting (Android)
date: 2019-07-18 16:38:05 +0300
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

NOTE: all emulator options are omitted for brevity in examples below.

## Could not determine java version from '9.0.4'

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

## Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

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

## No such file or directory - /usr/local/share/android-sdk

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

## repositories.cfg could not be loaded

1. <https://askubuntu.com/questions/885658>

```sh
$ sdkmanager --list
Warning: File /Users/tap/.android/repositories.cfg could not be loaded.
```

**solution**

```sh
$ touch ~/.android/repositories.cfg
```

## truncated package paths in output from sdkmanager

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

## Qt library not found

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

**_UPDATE_**

after updating emulator to version 26.1.4.0 it's no longer necessary to cd to
emulator directory but still you cannot rely on `PATH` - use absolute path:

_~/.zshenv_:

```conf
alias avd='$ANDROID_HOME/tools/emulator -avd Nexus_5X_API_23_x86_64'
# `Qt library not found` again:
#alias avd='emulator -avd Nexus_5X_API_23_x86_64'
```

## Cannot find AVD system path

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

## The SDK directory does not exist

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

## Could not get unknown property 'MYAPP_RELEASE_STORE_FILE'

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

## DeviceException: No connected devices!

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

## Could not find com.facebook.react:react-native

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

## CANNOT TRANSLATE guest DNS ip

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

## no application launcher icon on home screen in emulator

1. <https://developer.android.com/guide/practices/ui_guidelines/icon_design_launcher.html>

application doesn't have launcher icon on home screen in emulator.

**solution**

`react-native run-android` installs application but doesn't add application
launcher icon on home screen - you can do it manually:

- go to applications
- click and hold application icon
- drag application icon to home screen

## Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

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

## You have not accepted the license agreements of the following SDK components

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

## Configuration with name 'default' not found.

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

## Manifest merger failed

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
> In some cases, it was the fault of the react-native libraries maintainers for
> using imports like + symbols that are ambiguous and generally take the last
> version available of the library and in some other cases was caused by the
> fact that the developer was actually using new features from the latest
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

AFAIU there can be either `com.android.support` or `androidx.*` libraries in the
output but not both.

the reason why AndroidX libraries leak into current application dependencies is
that many RN libraries use the latest versions of Google libraries instead of
specific ones, say (note `+` symbol):

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

### [RECOMMENDED] try upgrading RN packages

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
> androidx dependencies. react-native won't be ready for androidx until 0.60 so
> constrain the gcm version to 16.1.0 (the latest pre-androidx version).

```diff
  # https://github.com/react-native-community/react-native-device-info/pull/693/files#diff-7ae5a9093507568eabbf35c3b0665732

  dependencies {
    implementation "com.facebook.react:react-native:${safeExtGet('reactNativeVersion', '+')}"
-   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
+   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '16.1.0')}"
```

### fork RN packages

1. <https://github.com/react-native-community/react-native-device-info/pull/693>

if RN package is not actively maintained it might take a long time till it's
fixed, so it makes sense to fork and fix it by yourself (just like it's done in
the linked PR which is also mentioned previously).

NOTE: you can open a PR to the upstream repository from a fork.

### force use of specific versions of Google libraries

1. <https://github.com/facebook/react-native/issues/25292#issuecomment-502998885>
2. <https://medium.com/@suchydan/how-to-solve-google-play-services-version-collision-in-gradle-dependencies-ef086ae5c75f>
3. <https://github.com/facebook/react-native/issues/25307#issuecomment-503516458>
4. <https://medium.com/mindorks/avoiding-conflicts-in-android-gradle-dependencies-28e4200ca235>

in this solution it's not necessary to modify Google libraries but your own
_android/app/build.gradle_ file only.

perform these steps for each RN package which depends on AndroidX libraries
until there are no AndroidX dependencies (run `gradlew app:dependencies` to find
such RN packages as described above):

- find Google libraries in RN package dependencies

  libraries inside `com.google.android.gms` and `com.google.firebase` groups are
  affected:

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

  a rule of thumb is not to use June versions, the last but one versions will do
  in most cases.

  see [Maven repository](https://mvnrepository.com/) for the list of library
  versions and corresponding release dates.

  it might be necessary to force versions of both direct and transitive RN
  package dependencies (just like in example above) - inspect the output of
  `gradlew app:dependencies` command after adding all direct dependencies.

## Could not get unknown property 'android' for root project 'iceperkapp' of type org.gradle.api.Project

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

## ENOTEMPTY: directory not empty

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

## resource android:style/TextAppearance.Material.Widget.Button.Colored not found

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
