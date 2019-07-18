---
layout: post
title: React Native - Troubleshooting (iOS errors)
date: 2019-07-18 16:38:22 +0300
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

## Could not parse the simulator list output

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

## duplicate interface definition for class 'RCTView'

1. <https://github.com/shoutem/ui/issues/134>

| Xcode: `Issue navigator` → `Buildtime`

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
    ...
    "react-native": "^0.42.0",
    ...
    "react-native-linear-gradient": "^1.6.2"
    ...
  },
```

that is currently used version of `react-native-linear-gradient` package is not
even supposed to support this version of react-native.

there are 2 ways to solve the problem:

- manually edit files in `react-native-linear-gradient` package

  | Xcode: `Project navigator`

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
    ...
    "react-native-linear-gradient": "^2.0.0",
    ...
  },
  ```

  ```sh
  $ npm update react-native-linear-gradient
  ```

## Entry, ":CFBundleIdentifier", Does Not Exist

1. <https://github.com/facebook/react-native/issues/7308>

```sh
$ react-native run-ios
...
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

**solution**

error was gone after solving problem with `react-native-linear-gradient` package
(see above) and reinstalling all node modules.

## No bundle URL present

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

## Native module cannot be null

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

## excessive logging in device system log

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

## Domain: NSURLErrorDomain, Error Code: -1200

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

<!-- prettier-ignore -->
| Xcode: `Project navigator` → _\<MY_APP>/\<MY_APP>/Info.plist_

## Library not loaded: .../CoreSimulator

the error occurred after updating Xcode from 8.2.1 to 9.2.

```sh
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

## ld: library not found for -lFlurry-iOS-SDK

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

## <PBXGroup ...> attempted to initialize an object with an unknown UUID

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

## CocoaPods could not find compatible versions for pod "react-native-contacts"

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

## CocoaPods could not find compatible versions for pod "Google-Mobile-Ads-SDK"

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

## CocoaPods could not find compatible versions for pod "RNCAsyncStorage"

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

## Could not find iPhone 6 simulator

```sh
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (13ms)
Found Xcode workspace iceperkapp.xcworkspace

Could not find iPhone 6 simulator
```

**solution**

1. <https://github.com/facebook/react-native/issues/23282>

upgrade RN.

## Undefined symbol: \_OBJC_METACLASS\_\$\_RCTEventEmitter

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

  1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-without-cocoapods>

## Invalid bitcode version (Producer: '802.0.41.0_0' Reader: '800.0.42.1_0')

| Xcode: `Product` (top menu) → `Archive`

```
error: Invalid bitcode version (Producer: '802.0.42.0_0' Reader: '800.0.42.1_0')
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://stackoverflow.com/questions/43480556/xcode-8-2-1-error-invalid-bitcode-version-producer-802-0-41-0-0-reader>

either:

- _[RECOMMENDED]_ update Xcode to 8.3
- disable bitcode in `Build Settings` → `Build Options` → `Enable Bitcode`

## Multiple commands produce '...libyoga.a'

| Xcode: `Product` (top menu) → `Archive`

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
