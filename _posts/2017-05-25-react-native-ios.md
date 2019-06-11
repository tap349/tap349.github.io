---
layout: post
title: React Native - iOS
date: 2017-05-25 12:38:50 +0300
access: public
comments: true
categories: [react-native, ios]
---

<!-- more -->

* TOC
{:toc}
<hr>

installation
------------

1. <https://facebook.github.io/react-native/docs/getting-started.html>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

install prerequisites and react-native-cli:

```sh
$ brew install node watchman
$ npm install -g react-native-cli
```

install Xcode and Command Line Tools
(both have been already installed on my MacBook).

running
-------

NOTE: emulator for iOS is called Simulator (will be referred to as just
      emulator for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS server)

same considerations as for Android.

```sh
$ react-native start
```

### run application in emulator

```sh
$ react-native run-ios
```

this command:

- builds application
- starts emulator
- installs application

the first run might take a while since RN will build the whole Xcode project.

configuration
-------------

### Dvorak keyboard layout

by default emulator uses QWERTY as hardware keyboard layout - it
also means that all hotkeys (say, for Developer Menu) use it as well.

it's possible to change it to Dvorak inside emulator:

| emulator (OS): `Settings` → `General` → `Keyboard` → `Hardware Keyboard` → `English (United States)` → `Dvorak`

### Russian keyboard layout

| emulator menu: `Hardware` → `Keyboard` → `Use the Same Keyboard Language as macOS`

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

NOTE: hot reloading is recommended over live reload (the former is much faster).

| emulator (app) → `<D-d>` → `Enable Live Reload`

| emulator (app) → `<D-d>` → `Enable Hot Reloading`

### scroll with 3 fingers

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

### hide device bezels

| emulator menu: `Window` → [ ] `Show Device Bezels`

device bezels are shown by default since Xcode 9
(there was no such option at all before Xcode 9 AFAIK).

### [Xcode 9.3+] turn on pixel accurate mode

| emulator menu: `Window` → [x] `Pixel Accurate`

otherwise actual paddings, margins, etc. might be different from specified ones.

tips
----

### (how to) build application in Xcode

- open application workspace (_\<APP\_NAME>.xcworkspace_)

  1. <https://stackoverflow.com/a/21644948/3632318>

  > <https://stackoverflow.com/a/21644948/3632318>
  >
  > You can still open your project files separately, but it is likely their
  > targets won’t build because Xcode cannot resolve the dependencies unless
  > you open the workspace file.
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

  | Xcode: `Product` (top menu) → `Clean Build Folder`

- build application as an executable product

  | Xcode: `Product` (top menu) → `Build`

### (how to) link library with native dependencies manually

1. <http://facebook.github.io/react-native/docs/linking-libraries-ios#manual-linking>

sometimes `react-native link` fails to link library with native dependencies -
in this case it's necessary to link library manually.

### (how to) repair CocoaPods

`pod deintegrate` command removes CocoaPods from Xcode project - it's not
necessary to fix the problem in most cases.

=> try installing pods with `pod install` command first. remove CocoaPods
if it doesn't help only.

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
    # ...
  end
```

```sh
$ cd ios
$ pod install
```

### (how to) change screen resolution (scale) in emulator

1. <https://stackoverflow.com/questions/10481412>

| emulator menu: `Window` → `Scale`

or

| emulator (app):

- `<D-1>` - 100% (default)
- `<D-2>` - 75%
- `<D-3>` - 50%
- `<D-4>` - 33%
- `<D-5>` - 25%

also there must be a way to change default scale by setting a value for
appropriate preference key via `defaults write` as described in the link
above - but it didn't work for me (probably because Apple changes these
keys all the time).

### (how to) run another simulator

running another simulator means using another iPhone model - not another iOS
version.

by default iPhone 6 simulator is used.

***UPDATE (2019-06-11)***

by default iPhone X simulator is used.

#### switch from command line

- close currently running simulator - or else it will be used even if simulator
  of another iPhone model is started
- `$ react-native run-ios --simulator 'iPhone 5'`

NOTE: it's necessary to configure new simulator separately
      (see [configuration](#configuration)).

#### switch from inside simulator

| emulator menu: `Hardware` → `Device` → `iOS 10.2` → `iPhone 5`

- `$ react-native run-ios` (reinstall application)

### (how to) upload file to emulator

just drag any file from Finder onto emulator window - if it's an image it will
be automatically copied to Photos on iOS (I didn't experiment with other types
of files). moreover it will be copied to Photos regardless of what application
is currently opened.

### (how to) add iPhone 4s simulator

iPhone 4s model is not available by default - add it manually:

| Xcode: `Product` (top menu) → `Destination` → `Add Additional Simulators...`
| `Simulators` (tab) → `+` (icon in bottom left corner)
| `OS Version` (dropdown menu) → `Download more simulator runtimes...`

- download `iOS 9.3 Simulator` runtime (the last version supported by iPhone 4s)
- create simulator for iPhone 4s using downloaded runtime

### (how to) delete all application data

remove application inside emulator and install it again.

debugging
---------

1. <https://facebook.github.io/react-native/docs/debugging.html>

### print to device system log

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

### Developer Menu

| emulator (app): `<D-d>`

### Inspector

| emulator (app): `<D-d>` → `Show Inspector`

or

| emulator (app): `<D-i>` (toggle)

### device system log (`iOS device syslog`)

```sh
$ react-native log-ios
```

### reload application in emulator manually

| emulator (app): `<D-d>` → `Reload`

or

| emulator (app): `<D-r>`

### show touchable areas

| emulator (app): `<D-i>` → `Touchables` (bottom menu) → `<D-i>`

=> touchable areas are still marked with dotted line

### toggle software keyboard

| emulator menu: `Hardware` → `Keyboard` → `Toggle Software Keyboard`

or

| emulator (app): `<D-k>`

troubleshooting
---------------

### duplicate interface definition for class 'RCTView'

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
  _\<MY\_APP>/Libraries/BVLinearGradient.xcodeproj/BVLinearGradient/_:

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
`SIMCTL_CHILD_OS_ACTIVITY_MODE` environment variable in the shell before starting
application (<https://github.com/bradmartin/nativescript-videoplayer/issues/76>):

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

problem was solved by disabling forward secrecy support
for offending URL in _Info.plist_:

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

| Xcode: `Project navigator` → _\<MY\_APP>/\<MY\_APP>/Info.plist_

### Library not loaded: .../CoreSimulator

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

### ld: library not found for -lFlurry-iOS-SDK

build inside Xcode failed:

```
ld: warning: directory not found for option '-L/Users/tap/Library/Developer/Xcode/DerivedData/iceperkapp-agmwzlgvaynpvuadwqakyphzpfof/Build/Products/Debug-iphoneos/Flurry-iOS-SDK'
ld: library not found for -lFlurry-iOS-SDK
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

> <https://github.com/xxsnakerxx/react-native-flurry-analytics/issues/8>
>
> Try closing the Xcode project and instead open <project name>.xcworkspace.
> Since this is using Cocopods you need to use the workspace instead for
> everything to get resolved correctly.

see also the tip `(how to) build project in Xcode` on why to open workspace
rather than a project.

still library might be really missing - in case you haven't linked it or
automatic linking by `react-native link` failed.

see the tip `(how to) link library with native dependencies manually` on
how to link library manually in the latter case.

### <PBXGroup ...> attempted to initialize an object with an unknown UUID

```
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

```
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

```sh
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (13ms)
Found Xcode workspace iceperkapp.xcworkspace

Could not find iPhone 6 simulator
```

**solution**

1. <https://github.com/facebook/react-native/issues/23282>

upgrade RN.
