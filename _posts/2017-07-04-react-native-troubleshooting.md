---
layout: post
title: React Native - Troubleshooting
date: 2017-07-04 18:40:40 +0300
access: public
comments: true
categories: [react-native]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

see [npm/Yarn - Tips]({% post_url 2017-11-19-npm-yarn-tips %}) for `npm_reset`.

try resetting cache first before googling the error:

```sh
$ react-native start --reset-cache
```

## JS errors

{% include_relative 2019-07-18-react-native-troubleshooting-(js-errors).md %}

## iOS errors

{% include_relative 2019-07-18-react-native-troubleshooting-(ios-errors).md %}

## Android errors

{% include_relative 2019-07-18-react-native-troubleshooting-(android-errors).md %}

## errors after upgrading RN to 0.45.1

### Cannot find module X

```sh
$ react-native start
...
> node node_modules/react-native/local-cli/cli.js start

module.js:487
    throw err;
    ^

Error: Cannot find module 'xmldoc'
    at Function.Module._resolveFilename (module.js:485:15)
```

and similar errors when some module cannot be found.

**solution**

use yarn instead of npm - somehow it managed to install all required
dependencies:

```sh
$ rm package-lock.json
$ rm -rf node-modules
$ yarn install
$ yarn start
```

### DeviceInfo native module is not installed correctly

emulator window:

```
DeviceInfo native module is not installed correctly
```

**solution**

1. <https://github.com/rebeccahughes/react-native-device-info/issues/176>

rebuild application.

### Unhandled JS Exception: undefined is not an object

emulator window:

```
Unhandled JS Exception: undefined is not an object (evaluating 'PropTypes.shape')
```

**solution**

1. <https://github.com/jsierles/react-native-audio/issues/83>

first I tried to upgrade RN again:

```sh
$ react-native upgrade
...
You should consider using the new upgrade tool based on Git. It makes upgrades easier by resolving most conflicts automatically.
To use it:
- Go back to the old version of React Native
- Run "npm install -g react-native-git-upgrade"
- Run "react-native-git-upgrade"
See https://facebook.github.io/react-native/docs/upgrading.html
```

as advised I ran:

```sh
$ npm install -g react-native-git-upgrade
$ react-native-git-upgrade
```

`react-native-git-upgrade` command downgraded `react` package to another alpha
version and this is what most likely fixed the issue.

## errors after upgrading RN to 0.47.0

### Module JSTimersExecution is not a registered callable module

1. <https://stackoverflow.com/questions/45594935>

emulator window:

```
Module JSTimersExecution is not a registered callable module (calling callTimers)
```

**solution**

rebuild application.

### method does not override or implement a method from a supertype

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

## errors after upgrading RN to 0.52.1

### Error: While resolving module `react-native-vector-icons/Octicons`

```
$ react-native start
...
error: bundling failed: Error: While resolving module `react-native-vector-icons/Octicons`, the Haste package `react-native-vector-icons` was found. However the module `Octicons` could not be found within the package. Indeed, none of these files exist:

  * `<APP_DIR>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
  * `<APP_DIR>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons/index(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
```

**solution**

1. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-357405396>
2. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-358018994>

```sh
$ rm ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json
$ react-native start
```

it looks like this file is recreated after each application build so it makes
sense to add this command to `postinstall` script in _package.json_:

```diff
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start",
    "test": "jest",
    "eslint": "eslint",
    "flow": "flow",
+   "postinstall": "rm -f ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json"
  },
```

### fatal error: 'Flurry.h' file not found

```
$ react-native run-ios
...
<APP_DIR>/node_modules/react-native-flurry-analytics/ios/RNFlurryAnalytics.m:2:9: fatal error: 'Flurry.h' file not found
#import <Flurry.h>
        ^~~~~~~~~~
1 error generated.
```

**solution**

see [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}) for the tip
on how to repair CocoaPods.

### object file (.../libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)

```
$ react-native run-ios
...
ld: warning: object file (<APP_DIR>/ios/build/Build/Products/Debug-iphonesimulator/libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)
```

**solution**

1. <https://stackoverflow.com/a/32950454/3632318>

`react-native-autogrow-textinput` is built with `IPHONEOS_DEPLOYMENT_TARGET=9.3`
build setting but the rest of the project (the library is linked to later) has
been built with `IPHONEOS_DEPLOYMENT_TARGET=8.0` build setting => hence the
error.

the latest version of `react-native-autogrow-textinput` doesn't support iOS
Deployment Target (DT) lower than 9.3 but current DT in iOS project is 8.0 =>
change DT in iOS project to 9.3 as well:

| Xcode: `Build Settings` → `Deployment` → `iOS Deployment Target`

### Undefined symbols for architecture x86_64

```
$ react-native run-ios
...
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_RCTOneSignal", referenced from:
      objc-class-ref in AppDelegate.o
  "_OBJC_CLASS_$_RNSentry", referenced from:
      objc-class-ref in AppDelegate.o
ld: symbol(s) not found for architecture x86_64
```

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/18#issuecomment-287132994>
2. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-onesignal` and `react-native-sentry` libraries manually and
rebuild application.

### Cannot read property 'func' of undefined

error is shown in device system log and emulator window.

the error occurs when evaluating this line:

```javascript
onPress: React.PropTypes.func.isRequired;
```

**solution**

1. <https://stackoverflow.com/a/47878585/3632318>

> PropTypes has been moved to a separate package. Accessing React.PropTypes is
> no longer supported and will be removed completely in React 16.

import `PropTypes` from a separate `prop-types` package.

### Cannot read property 'appVersion' of undefined

error is shown in device system log and emulator window.

the error occurs when evaluating this line:

```javascript
// node_modules/react-native-device-info/deviceinfo.js

return RNDeviceInfo.appVersion;
```

**solution**

1. <https://github.com/rebeccahughes/react-native-device-info/issues/52#issuecomment-340212477>
2. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-device-info` library manually and rebuild application.

### Invariant Violation: Native component for "BVLinearGradient" does not exist

error in device system log and emulator window.

**solution**

1. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-linear-gradient` library manually and rebuild application.

### TypeError: Cannot read property 'isPickerShow' of undefined

error in device system log and emulator window.

**solution**

1. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-picker` library manually and rebuild application.

### Cannot read property 'showImagePicker' of undefined

error in device system log and emulator window.

**solution**

1. <https://github.com/react-community/react-native-image-picker/issues/712>

link `react-native-image-picker` library manually and rebuild application.

### TypeError: undefined is not an object (evaluating 'Contacts.getAll')

error in device system log and emulator window.

**solution**

link `react-native-contacts` library manually and rebuild application.

### Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'

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

### production release crashes on startup (in both Android and iOS devices)

**solution**

1. <https://github.com/facebook/react-native/issues/16567#issuecomment-340041357>
2. <https://github.com/facebook/react-native/issues/16542>

<https://github.com/magus/react-native-facebook-login/issues/279>:

> View.propTypes has been removed in 0.49

replace `View.propTypes.style` with `ViewPropTypes.style` in all components.

see also [React Native -
Android]({% post_url 2017-05-24-react-native-android %}) (`debugging` section)
on how to debug similar problems on Android.

## errors after upgrading RN from 0.52.1 to 0.59.9

### react-native-push-notification: Appears to be a git repo or submodule.

```
$ npm install
...
npm ERR! path <APP_DIR>/node_modules/react-native-push-notification
npm ERR! code EISGIT
npm ERR! git <APP_DIR>/node_modules/react-native-push-notification: Appears to be a git repo or submodule.
npm ERR! git     <APP_DIR>/node_modules/react-native-push-notification
npm ERR! git Refusing to remove it. Update manually,
npm ERR! git or move it out of the way first.
```

**solution**

1. <https://github.com/zo0r/react-native-push-notification/issues/1057>

```sh
$ rm -rf node_modules/*/.git
```

### incompatible types: ReadableArray cannot be converted to ReadableNativeArray

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

### constructor RNSentryPackage in class RNSentryPackage cannot be applied to given types

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

### Cannot fit requested classes in a single dex file

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

### The Google Mobile Ads SDK was initialized incorrectly

emulator:

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

### Invalid application ID

emulator:

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

### Plugin/Preset files are not allowed to export objects, only functions.

```
$ react-native start
...
Loading dependency graph, done.
error: bundling failed: Error: Plugin/Preset files are not allowed to export objects, only functions. In <APP_DIR>/node_modules/babel-preset-flow/lib/index.js
    at createDescriptor (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:178:11)
    at <APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:109:50
    at Array.map (<anonymous>)
    at createDescriptors (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:109:29)
    at createPresetDescriptors (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:101:10)
    at presets (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:47:19)
    at mergeChainOpts (<APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:320:26)
    at <APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:283:7
    at mergeExtendsChain (<APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:299:21)
```

**solution**

1. <https://github.com/facebook/flow/issues/7046#issuecomment-443397899>

try updating corresponding npm package:

```
$ npm uninstall babel-preset-flow
$ npm install --save-dev @babel/preset-flow
```

```diff
  // babel.config.js

  module.exports = {
-   presets: ['flow']
+   presets: ['@babel/preset-flow']
  };
```

### The 'decorators' plugin requires a 'decoratorsBeforeExport' option, whose value must be a boolean

```
$ react-native start
...
error: bundling failed: Error: The 'decorators' plugin requires a 'decoratorsBeforeExport' option, whose value must be a boolean. If you are migrating from Babylon/Babel 6 or want to use the old decorators proposal, you should use the 'decorators-legacy' plugin instead of 'decorators'.
    at validatePlugins (<APP_DIR>/node_modules/@babel/parser/lib/index.js:6093:13)
    at getParser (<APP_DIR>/node_modules/@babel/parser/lib/index.js:11286:5)
    at parse (<APP_DIR>/node_modules/@babel/parser/lib/index.js:11256:22)
    at parser (<APP_DIR>/node_modules/@babel/core/lib/transformation/normalize-file.js:170:34)
    at normalizeFile (<APP_DIR>/node_modules/@babel/core/lib/transformation/normalize-file.js:138:11)
    at runSync (<APP_DIR>/node_modules/@babel/core/lib/transformation/index.js:44:43)
    at transformSync (<APP_DIR>/node_modules/@babel/core/lib/transform.js:43:38)
    at Object.transform (<APP_DIR>/node_modules/metro-react-native-babel-transformer/src/index.js:202:20)
```

**solution**

1. <https://react-redux.js.org/introduction/basic-tutorial#connecting-the-components>

> <https://www.npmjs.com/package/babel-plugin-transform-decorators-legacy#babel--7x>
>
> This plugin is specifically for Babel 6.x. If you're using Babel 7, this
> plugin is not for you. Babel 7's @babel/plugin-proposal-decorators officially
> supports the same logic that this plugin has, but integrates better with Babel
> 7's other plugins.

but after having tried all possible combinations of `decoratorsBeforeExport` and
`legacy` options I still couldn't make new decorators work. so:

> <https://github.com/reduxjs/react-redux/issues/1162#issuecomment-453847059>
>
> We do not recommend using connect as a decorator, because the spec is
> unstable.

=> it's easier not to use decorators at all (and remove all related packages),
the more especially as it's a recommended approach.

### TypeError: undefined is not an object (evaluating 'props.getItem')

```
$ adb logcat
...
06-10 00:18:39.077 10262 10367 E ReactNativeJS: TypeError: undefined is not an object (evaluating 'props.getItem')
06-10 00:18:39.077 10262 10367 E ReactNativeJS:
06-10 00:18:39.077 10262 10367 E ReactNativeJS: This error is located at:
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in FlatList (at YellowBoxList.js:87)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in RCTView (at View.js:45)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in View (at YellowBoxList.js:79)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in YellowBoxList (at YellowBox.js:104)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in YellowBox (at AppContainer.js:93)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in RCTView (at View.js:45)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in View (at AppContainer.js:115)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in AppContainer (at renderApplication.js:34)
06-10 00:18:39.090 10262 10367 E ReactNativeJS: TypeError: undefined is not an object (evaluating 'props.getItem')
```

**solution**

1. <https://github.com/facebook/react-native/issues/21154#issuecomment-439348692>

in this case and in case of many other weird JS errors first try resetting
cache:

```
$ react-native start --reset-cache
```

### We ran "xcodebuild" command but it exited with error code 65

```
$ react-native run-ios
...
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening iceperkapp.xcworkspace
```

**solution**

this can be anything so:

> To debug build logs further, consider building your app with Xcode.app, by
> opening iceperkapp.xcworkspace

### Duplicate module name: react-native

```
$ react-native start
...
Loading dependency graph...(node:10228) UnhandledPromiseRejectionWarning: Error: jest-haste-map: Haste module naming collision:
  Duplicate module name: react-native
  Paths: <APP_DIR>/ios/Pods/React/package.json collides with <APP_DIR>/node_modules/react-native/package.json

This error is caused by `hasteImpl` returning the same name for different files.
    at setModule (<APP_DIR>/node_modules/jest-haste-map/build/index.js:569:17)
    at workerReply (<APP_DIR>/node_modules/jest-haste-map/build/index.js:641:9)
    at processTicksAndRejections (internal/process/task_queues.js:89:5)
    at async Promise.all (index 83)
(node:10228) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 2)
(node:10228) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
(node:10228) UnhandledPromiseRejectionWarning: Error: jest-haste-map: Haste module naming collision:
  Duplicate module name: react-native
  Paths: <APP_DIR>/ios/Pods/React/package.json collides with <APP_DIR>/node_modules/react-native/package.json
```

**solution**

1. <https://facebook.github.io/react-native/docs/integration-with-existing-apps#configuring-cocoapods-dependencies>
2. <https://github.com/react-native-community/react-native-svg/issues/621#issuecomment-473346430>

incorrect version of `React` pod was installed into _ios/Pods/_ as a dependency
of `RNCAsyncStorage` pod:

```ruby
# node_modules/@react-native-community/async-storage/RNCAsyncStorage.podspec

Pod::Spec.new do |s|
  s.name         = "RNCAsyncStorage"
  # ...

  s.dependency 'React'
end
```

there are 2 problems here:

- version of `React` pod is incorrect (0.11.0 but should be 0.59.9)
- `React` pod shouldn't be installed into _ios/Pods/_ at all (see below why)

now fix both problems:

- configure CocoaPods dependencies correctly

  see [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}) for the
  tip on how to configure CocoaPods dependencies.

- remove `React` pod from _ios/Pods/_ directory

  since `RNCAsyncStorage` pod has `path` option in _ios/Podfile_ (which points
  to corresponding package location in _node_modules/_ directory), it shouldn't
  be installed into _ios/Pods/_ - as well as its dependencies (that is `React`
  pod).

  but directory for `React` pod is still there and _ios/Pods/React/package.json_
  collides with _node_modules/react-native/package.json_ as both have the same
  `name` field - `react-native`.

  it happened because `pod install` doesn't remove unused pods from _ios/Pods/_
  => it should be done manually by deintegrating your project:

  ```
  $ cd ios
  $ pod deintegrate
  Deintegrating `iceperkapp.xcodeproj`
  Deleted 1 'Copy Pods Resources' build phases.
  Deleted 1 'Check Pods Manifest.lock' build phases.
  - libPods-iceperkapp.a
  - Pods-iceperkapp.debug.xcconfig
  - Pods-iceperkapp.release.xcconfig
  Deleting Pod file references from project
  - libPods-iceperkapp-tvOSTests.a
  - libPods-iceperkappTests.a
  Deleted 1 empty `Pods` groups from project.
  Removing `Pods` directory.

  Project has been deintegrated. No traces of CocoaPods left in project.
  Note: The workspace referencing the Pods project still remains.
  ```

  or just by removing _ios/Pods/_ directory:

  ```sh
  $ rm -rf ios/Pods
  ```

  finally install pods again:

  ```sh
  $ cd ios
  $ pod install
  ```

### no known class method for selector 'didReceiveRemoteNotification:'

build inside Xcode failed:

```
<APP_DIR>/ios/iceperkapp/AppDelegate.m:71:17: error: no known class method for selector 'didReceiveRemoteNotification:'
  [RCTOneSignal didReceiveRemoteNotification:notification];
                ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/421#issuecomment-373877766>

```diff
  // ios/iceperkapp/AppDelegate.m

- // Required for the notification event.
- - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)notification {
-   [RCTOneSignal didReceiveRemoteNotification:notification];
- }
-
```

### node_modules/sentry-cli-binary/bin/sentry-cli: No such file or directory

build inside Xcode failed:

```
/Users/tap/Library/Developer/Xcode/DerivedData/iceperkapp-fqktxncrxxusoqcuegfuetzddexv/Build/
  Intermediates.noindex/iceperkapp.build/Debug-iphonesimulator/iceperkapp.build/
  Script-00DD1BFF1BD5951E006B06BC.sh: line 4: ../node_modules/sentry-cli-binary/bin/sentry-cli:
  No such file or directory
Command PhaseScriptExecution failed with a nonzero exit code
```

**solution**

find and replace all occurrences of `sentry-cli-binary` with `@sentry/cli` in
your project.

### The iOS Simulator deployment target is set to 7.0

```
$ react-native run-ios
...
info warning: The iOS Simulator deployment target is set to 7.0, but the range of supported deployment target versions for this platform is 8.0 to 12.2.99. (in target 'InAppUtils')
```

**solution**

1. <https://github.com/CocoaPods/CocoaPods/issues/8069>

> <https://github.com/react-native-community/react-native-maps/issues/2638#issue-393483116>
>
> It appears the newest version of Xcode doesn't support a deployment target
> less than 8.0.

> <https://github.com/CocoaPods/CocoaPods/issues/7314#issuecomment-422368382>
>
> if the target is built for 9.0, then the Pods should build for 9.0, not for
> 8.0.
>
> It's nice to want to support more stuff, but it should come as fixes to the
> actual CocoaPods gem, not as complex workarounds in people's Podfile.

> <https://github.com/CocoaPods/CocoaPods/issues/7314#issuecomment-494659918>
>
> platform
>
> Specifies the platform for which a static library should be built, but NOT the
> deployment target for which it should be built (the number in platform :ios,
> '9.0' is only to find a compatible version of the pod). The actual deployment
> target will be lower than specified, will have unused symbols and will cause
> you a lot of headache.

```ruby
# node_modules/react-native-in-app-utils/react-native-in-app-utils.podspec

Pod::Spec.new do |s|
  # ...
  s.platform        = :ios, "7.0"
end
```

```ruby
# ios/Podfile

platform :ios, '9.0'
```

Podspecs and Podfile both specify deployment target for which a static library
should be built. still deployment target in Podfile is not the minimum allowed
deployment target => deployment target from Podspec might be used which will
cause warnings like above if that deployment target is not supported by Xcode.

=> ignore this warning for now.

### withRef is removed

emulator window:

```
Unhandled JS Exception: withRef is removed. To access the wrapped instance,
use a ref on the connected component
```

**solution**

1. <https://react-redux.js.org/api/connect#forwardref-boolean>

> <https://github.com/reduxjs/react-redux/releases/tag/v6.0.0>
>
> The withRef option to connect has been replaced with forwardRef. If
> {forwardRef : true} has been passed to connect, adding a ref to the connected
> wrapper component will actually return the instance of the wrapped component.

> <https://github.com/reduxjs/react-redux/issues/1291#issuecomment-494188070>
>
> Basically, change withRef to forwardRef, and then delete the
> .getWrappedInstance() part of componentRef.getWrappedInstance(). componentRef
> is your own component now.

say:

```diff
  // MyComponent.js

  export default connect(
    mapStateToProps,
    null,
    null,
-   {withRef: true},
+   {forwardRef: true},
  )(MyComponent);
```

usage:

```diff
  <MyComponent ref={ref => this._myComponentRef = ref} />

  // ...
- this._myComponentRef.getWrappedInstance().save();
+ this._myComponentRef.save();
```
