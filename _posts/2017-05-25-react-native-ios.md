---
layout: post
title: React Native - iOS
date: 2017-05-25 12:38:50 +0300
access: public
comments: true
categories: [react-native, ios]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## installation

1. <https://facebook.github.io/react-native/docs/getting-started.html>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

install prerequisites and react-native-cli:

```sh
$ brew install node watchman
$ npm install -g react-native-cli
```

install Xcode and Command Line Tools (both have been already installed on my
MacBook).

## running

NOTE: emulator for iOS is called Simulator (will be referred to as just emulator
for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS server)

same considerations as for Android.

```sh
$ react-native start
```

### start emulator and run application in emulator

```sh
$ react-native run-ios
```

this command:

- starts emulator
- builds application
- installs application

the first run might take a while since RN will build the whole Xcode project.

## configuration

### Dvorak keyboard layout

by default emulator uses QWERTY as hardware keyboard layout - it also means that
all hotkeys (say, for Developer Menu) use it as well.

it's possible to change it to Dvorak inside emulator:

| emulator (OS)                                                                    |
| -------------------------------------------------------------------------------- |
| `Settings` → `General` → `Keyboard` → `Hardware Keyboard` → `English` → `Dvorak` |

### Russian keyboard layout

| emulator                                                                       |
| ------------------------------------------------------------------------------ |
| `Hardware` (top menu) → `Keyboard` → `Use the Same Keyboard Language as macOS` |

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

NOTE: hot reloading is recommended over live reload (the former is much faster).

| emulator                       |
| ------------------------------ |
| `<D-d>` → `Enable Live Reload` |

| emulator                         |
| -------------------------------- |
| `<D-d>` → `Enable Hot Reloading` |

### scroll with 3 fingers

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

### hide device bezels

| emulator                                       |
| ---------------------------------------------- |
| `Window` (top menu) → [ ] `Show Device Bezels` |

device bezels are shown by default since Xcode 9 (there was no such option at
all before Xcode 9 AFAIK).

### [Xcode 9.3+] turn on pixel accurate mode

| emulator                                   |
| ------------------------------------------ |
| `Window` (top menu) → [x] `Pixel Accurate` |

otherwise actual paddings, margins, etc. might be different from specified ones.

## tips

### (how to) open application workspace in Xcode

1. <https://facebook.github.io/react-native/blog/2019/07/03/version-60#cocoapods-by-default>

```sh
$ xed ios
```

### (how to) clean project

1. <https://stackoverflow.com/a/48019133/3632318>

```sh
$ cd ios
$ xcodebuild clean
```

### (how to) build application in Xcode

- open application workspace (_\<APP_NAME>.xcworkspace_)

  > <https://stackoverflow.com/a/21644948/3632318>
  >
  > You can still open your project files separately, but it is likely their
  > targets won’t build because Xcode cannot resolve the dependencies unless you
  > open the workspace file.
  >
  > Projects contain files (code/resouces), settings, and targets that build
  > products from those files and settings. Workspaces contain projects which
  > can reference each other.
  >
  > I think projects are sufficient in most cases. Don’t use workspaces unless
  > there’s a specific reason.
  >
  > CocoaPods, which automatically handles 3rd party libraries for you, uses
  > workspaces. Therefore, you have to use them, too, when you use CocoaPods.

  => open workspace (not a project) when building application in Xcode.

- clean build folder (just in case)

  | Xcode                                       |
  | ------------------------------------------- |
  | `Product` (top menu) → `Clean Build Folder` |

- build application as an executable product

  | Xcode                          |
  | ------------------------------ |
  | `Product` (top menu) → `Build` |

### (how to) link library with native dependencies manually

1. <http://facebook.github.io/react-native/docs/linking-libraries-ios#manual-linking>

sometimes `react-native link` fails to link library with native dependencies -
in this case it's necessary to link library manually.

### (how to) repair CocoaPods

`pod deintegrate` command removes CocoaPods from Xcode project - it's not
necessary to fix the problem in most cases.

=> try installing pods with `pod install` command first. remove CocoaPods if it
doesn't help only.

```sh
$ gem install cocoapods
$ cd ios
$ pod deintegrate
$ pod repo update
$ pod install
```

### (how to) configure CocoaPods dependencies

1. <https://facebook.github.io/react-native/docs/integration-with-existing-apps#configuring-cocoapods-dependencies>

```diff
  # ios/Podfile

  target 'iceperkapp' do
    # Uncomment the next line if you're using Swift or would like to use dynamic frameworks
    # use_frameworks!

+   pod 'React', path: '../node_modules/react-native', subspecs: [
+     'Core',
+     'CxxBridge', # Include this for RN >= 0.47
+     'DevSupport', # Include this to enable In-App Devmenu if RN >= 0.43
+     'RCTText',
+     'RCTNetwork',
+     'RCTWebSocket', # Needed for debugging
+     'RCTAnimation', # Needed for FlatList and animations
+     'RCTImage',
+     'RCTPushNotification'
+   ]
+   # Explicitly include Yoga if you are using RN >= 0.42.0
+   pod 'yoga', path: '../node_modules/react-native/ReactCommon/yoga'

+   # Third party deps podspec link
+   pod 'DoubleConversion', podspec: '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
+   pod 'glog', podspec: '../node_modules/react-native/third-party-podspecs/glog.podspec'
+   pod 'Folly', podspec: '../node_modules/react-native/third-party-podspecs/Folly.podspec'

    # Pods for my_app
    pod 'Flurry-iOS-SDK/FlurrySDK'
    pod 'Google-Mobile-Ads-SDK'
    pod 'RNCAsyncStorage', path: '../node_modules/@react-native-community/async-storage'
    pod 'RNSVG', path: '../node_modules/react-native-svg'
    pod 'RNVectorIcons', path: '../node_modules/react-native-vector-icons'
    # ...
  end
```

for some reason including `Picker` pod from `react-native-picker` package causes
build to fail even though it's recommended in official docs.

```sh
$ cd ios
$ pod install
```

### (how to) change screen resolution (scale) in emulator

1. <https://stackoverflow.com/questions/10481412>

| emulator                      |
| ----------------------------- |
| `Window` (top menu) → `Scale` |

or

| emulator |
| -------- |
|          |

- `<D-1>` - 100% (default)
- `<D-2>` - 75%
- `<D-3>` - 50%
- `<D-4>` - 33%
- `<D-5>` - 25%

also there must be a way to change default scale by setting a value for
appropriate preference key via `defaults write` as described in the link above -
but it didn't work for me (probably because Apple changes these keys all the
time).

### (how to) run another simulator

running another simulator means using another iPhone model - not another iOS
version.

by default iPhone 6 simulator is used.

**_UPDATE (2019-06-11)_**

by default iPhone X simulator is used.

#### switch from command line

- close currently running simulator - or else it will be used even if simulator
  of another iPhone model is started
- `$ react-native run-ios --simulator 'iPhone 5'`

NOTE: configure new simulator separately (see [configuration](#configuration)).

#### switch from inside simulator

| emulator                                                   |
| ---------------------------------------------------------- |
| `Hardware` (top menu) → `Device` → `iOS 10.2` → `iPhone 5` |

- `$ react-native run-ios` (reinstall application)

### (how to) upload file to emulator

just drag any file from Finder onto emulator window - if it's an image it will
be automatically copied to Photos on iOS (I didn't experiment with other types
of files). moreover it will be copied to Photos regardless of what application
is currently opened.

### (how to) add iPhone 4s simulator

iPhone 4s model is not available by default - add it manually:

| Xcode                                                                 |
| --------------------------------------------------------------------- |
| `Product` (top menu) → `Destination` → `Add Additional Simulators...` |
| `Simulators` (tab) → `+` (icon in bottom left corner)                 |
| `OS Version` (dropdown menu) → `Download more simulator runtimes...`  |

- download `iOS 9.3 Simulator` runtime (the last version supported by iPhone 4s)
- create simulator for iPhone 4s using downloaded runtime

### (how to) delete all application data

remove application inside emulator and install it again.

## debugging

1. <https://facebook.github.io/react-native/docs/debugging.html>

### print to device system log

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

### Developer Menu

| emulator |
| -------- |
| `<D-d>`  |

### Inspector

| emulator                   |
| -------------------------- |
| `<D-d>` → `Show Inspector` |

or

| emulator         |
| ---------------- |
| `<D-i>` (toggle) |

### device system log (`iOS device syslog`)

```sh
$ react-native log-ios
```

### reload application in emulator manually

| emulator           |
| ------------------ |
| `<D-d>` → `Reload` |

or

| emulator |
| -------- |
| `<D-r>`  |

### show touchable areas

| emulator                                       |
| ---------------------------------------------- |
| `<D-i>` → `Touchables` (bottom menu) → `<D-i>` |

=> touchable areas are still marked with dotted line

### toggle software keyboard

| emulator                                                        |
| --------------------------------------------------------------- |
| `Hardware` (top menu) → `Keyboard` → `Toggle Software Keyboard` |

or

| emulator |
| -------- |
| `<D-k>`  |

## troubleshooting

### [0.52.1] fatal error: 'Flurry.h' file not found

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

### [0.52.1] object file (.../libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)

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

| Xcode                                                     |
| --------------------------------------------------------- |
| `Build Settings` → `Deployment` → `iOS Deployment Target` |

### [0.52.1] Undefined symbols for architecture x86_64

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

### [0.59.9] We ran "xcodebuild" command but it exited with error code 65

```
$ react-native run-ios
...
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening iceperkapp.xcworkspace
```

**solution**

this can be anything so:

> To debug build logs further, consider building your app with Xcode.app, by
> opening iceperkapp.xcworkspace

### [0.59.9] Duplicate module name: react-native

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

### [0.59.9] no known class method for selector 'didReceiveRemoteNotification:'

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

### [0.59.9] node_modules/sentry-cli-binary/bin/sentry-cli: No such file or directory

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

### [0.59.9] The iOS Simulator deployment target is set to 7.0

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

### Could not parse the simulator list output

the error occurs after new Xcode has been downloaded (9.3 in my case):

```
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (43ms)
Found Xcode workspace iceperkapp.xcworkspace
dyld: Symbol not found: _SimDeviceBootKeyDisabledJobs
  Referenced from: /Applications/Xcode.app/Contents/Developer/usr/bin/simctl
  Expected in: /Library/Developer/PrivateFrameworks/CoreSimulator.framework/Versions/A/CoreSimulator
 in /Applications/Xcode.app/Contents/Developer/usr/bin/simctl

Could not parse the simulator list output
```

**solution**

1. <https://joelennon.com/resolving-could-not-parse-the-simulator-list-output-when-running-react-native-apps-on-ios-simulator>

launch Xcode and install additional required components when prompted.

### duplicate interface definition for class 'RCTView'

1. <https://github.com/shoutem/ui/issues/134>

| Xcode                                  |
| -------------------------------------- |
| `Issue navigator` (icon) → `Buildtime` |

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

**solution**

outdated version of `react-native-linear-gradient` package was used (it provides
`LinearGradient` component).

<https://github.com/react-native-community/react-native-linear-gradient>:

> Version 2.0 supports react-native >= 0.40.0

now in _package.json_:

```json
  "dependencies": {
    "react-native": "^0.42.0",
    "react-native-linear-gradient": "^1.6.2",
  },
```

that is currently used version of `react-native-linear-gradient` package is not
even supposed to support this version of react-native.

there are 2 ways to solve the problem:

- manually edit files in `react-native-linear-gradient` package

  | Xcode               |
  | ------------------- |
  | `Project navigator` |

  replace old imports with new ones in all files inside
  _\<MY_APP>/Libraries/BVLinearGradient.xcodeproj/BVLinearGradient/_:

  ```objc
  // old import
  //#import "RCTView.h"
  // new import
  #import <React/RCTView.h>
  ```

- update `react-native-linear-gradient` package

  _package.json_:

  ```json
  "dependencies": {
    "react-native-linear-gradient": "^2.0.0",
  },
  ```

  ```sh
  $ npm update react-native-linear-gradient
  ```

### Entry, ":CFBundleIdentifier", Does Not Exist

1. <https://github.com/facebook/react-native/issues/7308>

```sh
$ react-native run-ios
...
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

**solution**

error was gone after solving problem with `react-native-linear-gradient` package
(see above) and reinstalling all node modules.

### No bundle URL present

1. <https://github.com/facebook/react-native/issues/12754>

emulator window:

```
No bundle URL present.

Make sure you're running a packager or have included a .jsbundle file in your
application bundle.
```

**solution**

it looks like application is run in emulator before packager is started.

run application again without closing emulator:

```sh
$ react-native run-ios
```

NOTE: the error usually occurs after running application in Android emulator.

### Native module cannot be null

1. <https://github.com/zo0r/react-native-push-notification/issues/160>
2. <https://github.com/zo0r/react-native-push-notification/issues/279>
3. <http://facebook.github.io/react-native/docs/pushnotificationios.html>
4. <http://facebook.github.io/react-native/docs/linking-libraries-ios.html>

packager log:

```sh
Native module cannot be null.
```

**solution**

the error occurred after installing `react-native-push-notification` package,
linking its native dependencies and trying to launch application:

```sh
$ npm install --save react-native-push-notification
$ react-native link react-native-push-notification
$ react-native run-ios
```

but it has turned out `PushNotificationIOS` library has not been linked to my
iOS project automatically (note there's no message about linking `iOS module`):

```sh
$ react-native link react-native-push-notification
Scanning 587 folders for symlinks in <APP_DIR>/node_modules (6ms)
npm-install info Android module react-native-push-notification is already linked
```

solution is to link `PushNotificationIOS` library manually as instructed in
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html).

### excessive logging in device system log

that is when running `react-native log-ios`.

**solution**

1. <https://stackoverflow.com/questions/37800790>
2. <https://github.com/bradmartin/nativescript-videoplayer/issues/76>

setting `OS_ACTIVITY_MODE` and `OS_ACTIVITY_DT_MODE` environment variables in
Xcode project had no effect (<https://stackoverflow.com/questions/37800790>).

the only thing that helped to reduce the amount of logging output was exporting
`SIMCTL_CHILD_OS_ACTIVITY_MODE` environment variable before starting application
(<https://github.com/bradmartin/nativescript-videoplayer/issues/76>):

```sh
$ SIMCTL_CHILD_OS_ACTIVITY_MODE="disable" react-native run-ios
```

or else this variable can be exported for all shells in _~/.zshenv_:

```zsh
export SIMCTL_CHILD_OS_ACTIVITY_MODE="disable"
```

### Domain: NSURLErrorDomain, Error Code: -1200

`WebView` component's content when trying to open HTTPS URL:

```
Error loading page

Domain: NSURLErrorDomain
Error Code: -1200
Description: An SSL error has occurred and a secure connection to the server cannot be made
```

device system log:

```
<Warning>: NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9824)
<Warning>: 'Encountered an error loading page', { domain: 'NSURLErrorDomain',
  loading: false,
  description: 'An SSL error has occurred and a secure connection to the server cannot be made.',
  target: 398,
  code: -1200,
  canGoBack: false,
  title: '',
  url: '',
  canGoForward: false }
```

**solution**

1. <https://stackoverflow.com/a/30882037/3632318>
2. <https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/>
3. <https://github.com/vinhnx/iOS-notes/issues/1>

problem was solved by disabling forward secrecy support for offending URL in
_Info.plist_:

```diff
  ...
  <key>NSAppTransportSecurity</key>
  <dict>
    <key>NSExceptionDomains</key>
    <dict>
      <key>localhost</key>
      <dict>
        <key>NSExceptionAllowsInsecureHTTPLoads</key>
        <true/>
      </dict>
+     <key>acs.***.ru</key>
+     <dict>
+       <key>NSTemporaryExceptionRequiresForwardSecrecy</key>
+       <false/>
+     </dict>
    </dict>
  </dict>
  ...
```

or else edit _Info.plist_ in Xcode:

| Xcode                                                  |
| ------------------------------------------------------ |
| `Project navigator` → _\<MY_APP>/\<MY_APP>/Info.plist_ |

### Library not loaded: .../CoreSimulator

the error occurred after updating Xcode from 8.2.1 to 9.2.

```
$ react-native run-ios
...
instruments[2536:17017] [MT] DVTPlugInLoading: Failed to load code for plug-in com.apple.xray.discovery.mobiledevice (/Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin), error = Error Domain=NSCocoaErrorDomain Code=3587 "dlopen_preflight(/Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/MacOS/XRMobileDeviceDiscoveryPlugIn): Library not loaded: /Library/Developer/PrivateFrameworks/CoreSimulator.framework/Versions/A/CoreSimulator
  Referenced from: /Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/MacOS/XRMobileDeviceDiscoveryPlugIn
  Reason: image not found"
...
```

**solution**

1. <https://github.com/facebook/react-native/issues/14342>

install additional components - you'll be prompted to install these components
when you first start Xcode after updating it.

### ld: library not found for -lFlurry-iOS-SDK

build inside Xcode failed:

```
ld: warning: directory not found for option '-L/Users/tap/Library/Developer/Xcode/DerivedData/iceperkapp-agmwzlgvaynpvuadwqakyphzpfof/Build/Products/Debug-iphoneos/Flurry-iOS-SDK'
ld: library not found for -lFlurry-iOS-SDK
```

**solution**

> <https://github.com/xxsnakerxx/react-native-flurry-analytics/issues/8>
>
> Try closing the Xcode project and instead open <project name>.xcworkspace.
> Since this is using Cocopods you need to use the workspace instead for
> everything to get resolved correctly.

see also the tip `(how to) build application in Xcode` on why to open workspace
rather than a project when building application in Xcode.

still library might be really missing - in case you haven't linked it or
automatic linking by `react-native link` failed.

see the tip `(how to) link library with native dependencies manually` on how to
link library manually in the latter case.

### <PBXGroup ...> attempted to initialize an object with an unknown UUID

```
$ cd ios
$ pod install
...
[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `EEE09AF85CBC4DA8A7C4E137` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.

[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `9DEE1362370047E5817C99E0` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.
...
```

**solution**

1. <https://github.com/CocoaPods/CocoaPods/issues/1822#issuecomment-304815540>

these warnings might be gone after running `pod install` command because it
updates _ios/iceperkapp.xcodeproj/project.pbxproj_ fixing or removing unknown
UUIDs along the way:

```sh
$ cd ios
$ pod install
```

see the tip on how to repair CocoaPods if it doesn't help.

### CocoaPods could not find compatible versions for pod "react-native-contacts"

```sh
$ cd ios
$ pod install
...
[!] CocoaPods could not find compatible versions for pod "react-native-contacts":
  In Podfile:
    react-native-contacts (from `../node_modules/react-native-contacts`)

Specs satisfying the `react-native-contacts (from `../node_modules/react-native-contacts`)` dependency were found, but they required a higher minimum deployment target.
```

**solution**

<https://facebook.github.io/react-native/docs/linking-libraries-ios.html>:

> If your iOS project is using CocoaPods (contains Podfile) and linked library
> has podspec file, then react-native link will link library using Podfile.

pods were added to _ios/Podfile_ after linking corresponding libraries:

```diff
  target 'iceperkapp-tvOSTests' do
    inherit! :search_paths
    # Pods for testing
+   pod 'react-native-contacts', :path => '../node_modules/react-native-contacts'
+
+   pod 'RNDeviceInfo', :path => '../node_modules/react-native-device-info'
+
+   pod 'react-native-image-picker', :path => '../node_modules/react-native-image-picker'
+
+   pod 'BVLinearGradient', :path => '../node_modules/react-native-linear-gradient'
+
+   pod 'react-native-onesignal', :path => '../node_modules/react-native-onesignal'
+
+   pod 'Picker', :path => '../node_modules/react-native-picker'
+
+   pod 'SentryReactNative', :path => '../node_modules/react-native-sentry'
+
+   pod 'RNSVG', :path => '../node_modules/react-native-svg'
+
+   pod 'RNVectorIcons', :path => '../node_modules/react-native-vector-icons'
+
  end
```

remove `iceperkapp-tvOSTests` and `iceperkappTests` targets from _ios/Podfile_.

### CocoaPods could not find compatible versions for pod "Google-Mobile-Ads-SDK"

```sh
$ cd ios
$ pod install
...
Analyzing dependencies
[!] CocoaPods could not find compatible versions for pod "Google-Mobile-Ads-SDK":
  In snapshot (Podfile.lock):
    Google-Mobile-Ads-SDK (= 7.29.0)

  In Podfile:
    Google-Mobile-Ads-SDK

None of your spec sources contain a spec satisfying the dependencies: `Google-Mobile-Ads-SDK, Google-Mobile-Ads-SDK (= 7.29.0)`.

You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * mistyped the name or version.
 * not added the source repo that hosts the Podspec to your Podfile.

Note: as of CocoaPods 1.0, `pod repo update` does not happen on `pod install` by default.
```

**solution**

```sh
$ pod repo update
```

### CocoaPods could not find compatible versions for pod "RNCAsyncStorage"

```sh
$ cd ios
$ pod install
...
Fetching podspec for `RNCAsyncStorage` from `../node_modules/@react-native-community/async-storage`
[!] CocoaPods could not find compatible versions for pod "RNCAsyncStorage":
  In Podfile:
    RNCAsyncStorage (from `../node_modules/@react-native-community/async-storage`)

Specs satisfying the `RNCAsyncStorage (from `../node_modules/@react-native-community/async-storage`)` dependency were found, but they required a higher minimum deployment target.
```

**solution**

find minimum deployment target for `RNCAsyncStorage` pod:

```ruby
# node_modules/@react-native-community/async-storage/RNCAsyncStorage.podspec

Pod::Spec.new do |s|
  # ...
  s.platform     = :ios, "9.0"
end
```

use it as a global platform for your project in _ios/Podfile_:

```diff
  # ios/Podfile

- platform :ios, '8.0'
+ platform :ios, '9.0'
```

### Could not find iPhone 6 simulator

```
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (13ms)
Found Xcode workspace iceperkapp.xcworkspace

Could not find iPhone 6 simulator
```

**solution**

1. <https://github.com/facebook/react-native/issues/23282>

upgrade RN.

### Undefined symbol: \_OBJC_METACLASS\_\$\_RCTEventEmitter

1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-add-notification-service-extension>

build inside Xcode failed:

```
Undefined symbols for architecture arm64:
  "_OBJC_METACLASS_$_RCTEventEmitter", referenced from:
      _OBJC_METACLASS_$_RCTOneSignalEventEmitter in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_OBJC_CLASS_$_RCTEventEmitter", referenced from:
      _OBJC_CLASS_$_RCTOneSignalEventEmitter in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTRegisterModule", referenced from:
      +[RCTOneSignalEventEmitter load] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTRunningInAppExtension", referenced from:
      -[RCTOneSignalEventEmitter checkPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter requestPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter getPermissionSubscriptionState:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTSharedApplication", referenced from:
      -[RCTOneSignalEventEmitter checkPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter requestPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_OBJC_CLASS_$_RCTConvert", referenced from:
      objc-class-ref in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
ld: symbol(s) not found for architecture arm64
```

error occurred after adding notification service extension to Xcode project (see
the link).

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/368#issuecomment-441473734>

- install React Native SDK without Cocoapods

  1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-installing-with-cocoapods-ios>

  ```diff
    # ios/Podfile

    target 'iceperkapp' do
      # ...
  -   pod 'react-native-onesignal', path: '../node_modules/react-native-onesignal'
    end
  ```

- install Notification Service Extension without Cocoapods

  1. <https://documentation.onesignal.com/docs/react-native-sdk-setup-continued-without-cocoapods>

### Invalid bitcode version (Producer: '802.0.41.0_0' Reader: '800.0.42.1_0')

| Xcode                            |
| -------------------------------- |
| `Product` (top menu) → `Archive` |

```
error: Invalid bitcode version (Producer: '802.0.42.0_0' Reader: '800.0.42.1_0')
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://stackoverflow.com/questions/43480556/xcode-8-2-1-error-invalid-bitcode-version-producer-802-0-41-0-0-reader>

either:

- _[RECOMMENDED]_ update Xcode to 8.3
- disable bitcode in `Build Settings` → `Build Options` → `Enable Bitcode`

### Multiple commands produce '...libyoga.a'

| Xcode                            |
| -------------------------------- |
| `Product` (top menu) → `Archive` |

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

<!-- prettier-ignore -->
> <https://github.com/facebook/react-native/issues/20492#issuecomment-422958184>
>
> The following is needed to ensure the "archive" step works in XCode. It
> removes React & Yoga from the Pods project, as it is already included in the
> main project.
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

### 'Flurry.h' file not found

error occurs after switching to `react-native-flurry-sdk` npm package (from
`react-native-flurry-analytics`):

```
$ react-native run-ios
...
<APP_DIR>/node_modules/react-native-flurry-sdk/ios/ReactNativeFlurry/ReactNativeFlurry.m:23:9: fatal error: 'Flurry.h' file not found
...
<APP_DIR>/node_modules/react-native-flurry-sdk/ios/ReactNativeFlurry/ReactNativeFlurryConfigListener.h:23:9: fatal error: 'FConfig.h' file not found
```

**solution**

> <https://github.com/flurry/react-native-flurry-sdk/issues/14#issuecomment-517790903>
>
> If you are not using CocoaPods, please stick to react-native-flurry-sdk@3.7.0.

### duplicate symbol _OBJC_CLASS_$_ReactNativeFlurry

```
$ react-native run-ios
...
info duplicate symbol _OBJC_CLASS_$_ReactNativeFlurry in:
    <APP_DIR>/ios/build/iceperkapp/Build/Products/Debug-iphonesimulator/libReactNativeFlurry.a(ReactNativeFlurry.o)
    <APP_DIR>/ios/build/iceperkapp/Build/Products/Debug-iphonesimulator/libReactNativeFlurryWithMessaging.a(ReactNativeFlurry.o)
...
info ld: 920 duplicate symbols for architecture i386
```

**solution**

> <https://github.com/react-native-community/react-native-maps/issues/718#issuecomment-256842449>
>
> First I made sure that react-native link goes on without an error.

make sure you've linked the library:

```
$ react-native link react-native-flurry-sdk
info iOS module "react-native-flurry-sdk" is already linked
info Android module "react-native-flurry-sdk" is already linked
? Do you need to integrate Flurry Push? No
Flurry: libReactNativeFlurry.a is successfully linked to project.
```

### [0.60.5] duplicate symbol _OBJC_IVAR_$_SentryUser._extra

```
$ react-native run-ios
...
duplicate symbol _OBJC_IVAR_$_SentryUser._extra in:
    <APP_DIR>/ios/build/iceperkapp/Build/Products/Debug-iphonesimulator/libRNSentry.a(SentryUser.o)
    <APP_DIR>/ios/build/iceperkapp/Build/Products/Debug-iphonesimulator/Sentry/libSentry.a(SentryUser.o)
ld: 282 duplicate symbols for architecture i386
```

**solution**

CLI autolinking is used RN 0.60+ => you don't link libraries manually now.

in my case I replaced `react-native-sentry` with `@sentry/react-native` and
there were identical static library files but with different names in build
folder (_libRNSentry.a_ and _Sentry/libSentry.a_).

BTW running `Clean Build Folder` in Xcode didn't help - IDK what exactly it
cleans but _ios/build/iceperkapp/Build/Products/Debug-iphonesimulator/_ was
left intact.

removin iOS build folder helped:

```sh
$ rm -rf ios/build
```

### The Google Mobile Ads SDK was initialized incorrectly

application crashes on startup:

```
$ react-native run-ios
$ react-native log-ios
...
Aug 28 10:41:11 MacBook-Pro-Personal iceperkapp[96992]: <Error>: ***
Terminating app due to uncaught exception 'GADInvalidInitializationException',
reason: 'The Google Mobile Ads SDK was initialized incorrectly. Google AdMob
publishers should follow instructions here:
https://googlemobileadssdk.page.link/admob-ios-update-plist to include the
AppMeasurement framework, set the -ObjC linker flag, and set
GADApplicationIdentifier with a valid App ID. Google Ad Manager publishers
should follow instructions here:
https://googlemobileadssdk.page.link/ad-manager-ios-update-plist'
```

**solution**

`Google-Mobile-Ads-SDK` pod had been added to _ios/Podfile_ without specific
version so it was updated I removed _ios/Podfile.lock_ and install pods from
scratch.

> <https://developers.google.com/ad-manager/mobile-ads-sdk/ios/quick-start#update_your_infoplist>
>
> Declare that your app is an Ad Manager app by adding the GADIsAdManagerApp key
> with a boolean value YES to your app's Info.plist.
>
> This step is required as of Google Mobile Ads SDK version 7.42.0. Failure to
> add add this Info.plist entry results in a crash with the message: "The Google
> Mobile Ads SDK was initialized incorrectly."

```diff
  <!-- ios/iceperkapp/Info.plist -->

  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
  <plist version="1.0">
  <dict>
+   <key>GADIsAdManagerApp</key>
+   <true/>
```

### `useNativeDriver` is not supported because the native animated module is missing

emulator window:

```
Animated: `useNativeDriver` is not supported because the native animated module
is missing. Falling back to JS-based animation. To resolve this, add
`RCTAnimation` module to the app, or remove `useNativeDriver`. More info:
https://github.com/facebook/react-native/issues/11094#issuecomment-263240420
```

**solution**

> <https://github.com/facebook/react-native/issues/11094#issuecomment-263240420>
>
> drag and drop libRCTAnimation.a to Link Binary With Libraries

### Requiring unknown module "./locale/de"

application crashes on startup after changing system language to German (in
production and when run inside Xcode in release mode):

```
iceperkapp[3682:961450] Unhandled JS Exception: Requiring unknown module "./locale/de".
[error][tid:com.facebook.react.JavaScript] Requiring unknown module "./locale/de".
iceperkapp[3682:961413] Requiring unknown module "./locale/de".
iceperkapp[3682:961450] *** Terminating app due to uncaught exception
  'RCTFatalException: Unhandled JS Exception: Requiring unknown module
  "./locale/de".', reason: 'Unhandled JS Exception: Requiring unknown module
  "./locale/de"., stack:
```

**solution**

error message is self-explanatory and it's quite obvious how to fix the error.
what's interesting is that it couldn't be reproduced in emulator and on real
device while application was running in debug mode - only after switching to
release mode did application start to crash which allowed to see crash log in
Xcode and fix the error.

### Module RCTEventEmitter is not a registered callable module

emulator window:

```
Module RCTEventEmitter is not a registered callable module (calling receiveTouches)
```

**solution**

1. <https://stackoverflow.com/a/57108272/3632318>

```sh
$ react-native --reset-cache
```

### Module AppRegistry is not a registered callable module

emulator window:

```
Unhandled JS Exception: Module AppRegistry is not a registered callable module
(calling runApplication)
```

**solution**

> <https://github.com/facebook/react-native/issues/17724#issuecomment-360174492>
>
> This error often occurs when there is another problem with evaluating your JS
> bundle earlier. Basically, there’s another error that is literally covered up
> by the red box of this one.
>
> The developer experience could be better, but try checking the native device
> logs for errors to see what happened before this AppRegistry error.
