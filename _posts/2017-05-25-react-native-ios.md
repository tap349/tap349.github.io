---
layout: post
title: React Native - iOS
date: 2017-05-25 12:38:50 +0300
access: public
comments: true
categories: [react-native, ios]
---

<!-- more -->

<!-- prettier-ignore -->
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

by default emulator uses QWERTY as hardware keyboard layout - it also means
that all hotkeys (say, for Developer Menu) use it as well.

it's possible to change it to Dvorak inside emulator:

| emulator (OS): `Settings` → `General` → `Keyboard` → `Hardware Keyboard` → `English` → `Dvorak`

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

device bezels are shown by default since Xcode 9 (there was no such option at
all before Xcode 9 AFAIK).

### [Xcode 9.3+] turn on pixel accurate mode

| emulator menu: `Window` → [x] `Pixel Accurate`

otherwise actual paddings, margins, etc. might be different from specified ones.

tips
----

### (how to) clean project

1. <https://stackoverflow.com/a/48019133/3632318>

```sh
$ cd ios
$ xcodebuild clean
```

### (how to) build application in Xcode

- open application workspace (_\<APP\_NAME>.xcworkspace_)

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
    pod 'Flurry-iOS-SDK/FlurrySDK'
    pod 'Google-Mobile-Ads-SDK'
    pod 'RNCAsyncStorage', path: '../node_modules/@react-native-community/async-storage'
    pod 'RNSVG', path: '../node_modules/react-native-svg'
    pod 'RNVectorIcons', path: '../node_modules/react-native-vector-icons'
    # ...
  end
```

for some reason including `Picker` pod from `react-native-picker` package
causes build to fail even though it's recommended in official docs.

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
