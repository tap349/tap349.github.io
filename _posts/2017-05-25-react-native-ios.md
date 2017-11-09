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

## installation

1. <https://facebook.github.io/react-native/docs/getting-started.html>
2. <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

install prerequisites and react-native-cli:

```sh
$ brew install node watchman
$ npm install -g react-native-cli
```

install Xcode and Command Line Tools
(both have been already installed on my MacBook).

## running

NOTE: emulator for iOS is called Simulator (will be referred
      to as just emulator for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS server) (optional)

same considerations as for Android.

```sh
$ npm start
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

## configuration

### Dvorak keyboard layout

by default emulator uses QWERTY as hardware keyboard layout - it
also means that all hotkeys (say, for Developer Menu) use it as well.

it's possible to change it to Dvorak inside emulator:

`Settings` -> `General` -> `Keyboards` -> `Hardware Keyboard` -> `English` -> `Dvorak`

### Russian keyboard layout

emulator menu:

- `Hardware` -> `Keyboard` -> `Uses the Same Layout as macOS`
- `Hardware` -> `Keyboard` -> `Toggle Software Keyboard` (enable)

NOTE: it's not possible to input text in Russian without software keyboard.

### enable live/hot reloading

1. <https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

NOTE: hot reloading is recommended over live reload.

- `<D-d>` -> `Enable Live Reload`
- `<D-d>` -> `Enable Hot Reloading`

### scroll with 3 fingers

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

## tips

### change screen resolution (scale) in emulator

1. <https://stackoverflow.com/questions/10481412>

emulator menu: `Window` -> `Scale`

- `<D-1>` - 100% (default)
- `<D-2>` - 75%
- `<D-3>` - 50%
- `<D-4>` - 33%
- `<D-5>` - 25%

also there must be a way to change default scale by setting a value for
appropriate preference key via `defaults write` as described in the link
above - but it didn't work for me (probably because Apple changes these keys
all the time).

### run another simulator

running another simulator means using another iPhone model -
not another iOS version.

by default iPhone 6 simulator is used.

#### switch from command line

- close currently running simulator - or else it will be
  used even if simulator of another iPhone model is started
- `$ react-native run-ios --simulator 'iPhone 5'`

NOTE: it's necessary to configure new simulator separately
      (see [configuration](#configuration)).

#### switch from inside simulator

- emulator menu: `Hardware` -> `Device` -> `iOS 10.2` -> `iPhone 5`
- `$ react-native run-ios` (reinstall application)

### upload file to emulator

just drag any file from Finder onto emulator window - if it's an image it will
be automatically copied to Photos on iOS (I didn't experiment with other types
of files). moreover it will be copied to Photos regardless of what application
is currently opened.

## debugging

1. <https://facebook.github.io/react-native/docs/debugging.html>

### print to device system log

same as for [Android]({% post_url 2017-05-24-react-native-android %}).

### Developer Menu

`<D-d>`

### Inspector

- `<D-d>` -> `Show Inspector`
- `<D-i>` (toggle)

### device system log (`iOS device syslog`)

```sh
$ react-native log-ios
```

### reload application in emulator manually

- `<D-d>` -> `Reload` or
- `<D-r>`

### show touchable areas

- `<D-d>` -> `Show Inspector`
- click `Touchables` in bottom menu
- `<D-d>` -> `Hide Inspector`
- touchable areas are still marked with dotted line

### toggle software keyboard

emulator menu: `Hardware` -> `Keyboard` -> `Toggle Software Keyboard`

- `<D-k>`

## troubleshooting

### duplicate interface definition for class 'RCTView'

1. <https://github.com/shoutem/ui/issues/134>

Xcode -> `Issue navigator` -> `Buildtime`:

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

**solution**

outdated version of `react-native-linear-gradient` package was used
(it provides `LinearGradient` component).

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

that is currently used version of `react-native-linear-gradient` package is
not even supposed to support this version of react-native.

there are 2 ways to solve the problem:

- manually edit files in `react-native-linear-gradient` package

  Xcode -> `Project navigator`

  replace old imports with new ones in all files inside
  _\<MyApp\>/Libraries/BVLinearGradient.xcodeproj/BVLinearGradient/_:

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

error was gone after solving problem with `react-native-linear-gradient`
package (see above) and reinstalling all node modules.

### No bundle URL present

1. <https://github.com/facebook/react-native/issues/12754>

emulator window:

```
No bundle URL present.

Make sure you're running a packager or have included a .jsbundle file
in your application bundle.
```

**solution**

it looks like application is run in emulator before packager is started.

run application again without closing emulator:

```sh
$ react-native run-ios
```

NOTE: this error usually occurs after running application in Android emulator.

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

this error occured after installing `react-native-push-notification` package,
linking native libraries and trying to launch application:

```sh
$ npm install --save react-native-push-notification
$ react-native link react-native-push-notification
$ react-native run-ios
```

it has turned out `PushNotificationIOS` native library (has native dependencies)
from `react-native` package has not been linked automatically to my iOS project
when linking dependencies for `react-native-push-notification` package
(this is probably a bug in the latter):

```sh
$ react-native link react-native-push-notification
Scanning 587 folders for symlinks in /Users/tap/dev/complead/iceperkapp/node_modules (6ms)
npm-install info Android module react-native-push-notification is already linked
```

solution is to link `PushNotificationIOS` native library manually as instructed in
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html).

## excessive logging in device system log

that is when running `react-native log-ios`.

**solution**

1. <https://stackoverflow.com/questions/37800790>
2. <https://github.com/bradmartin/nativescript-videoplayer/issues/76>

setting `OS_ACTIVITY_MODE` and `OS_ACTIVITY_DT_MODE` environment
variables in Xcode project had no effect (link #1).

the only thing that helped to reduce the amount of logging output
was exporting `SIMCTL_CHILD_OS_ACTIVITY_MODE` environment variable
in the shell before starting application (link #2):

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

Xcode -> `Project navigator` -> _\<MyApp\>/\<MyApp\>/Info.plist_.
