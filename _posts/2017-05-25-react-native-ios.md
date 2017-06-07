---
layout: post
title: React Native - iOS
date: 2017-05-25 12:38:50 +0300
access: public
categories: [react-native, ios]
---

<!-- more -->

* TOC
{:toc}

## installation

- <https://facebook.github.io/react-native/docs/getting-started.html>
- <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

install prerequisites and react-native-cli:

```sh
$ brew install node watchman
$ npm install -g react-native-cli
```

install Xcode and Command Line Tools
(both have been already installed on my MacBook).

## running

NOTE: emulator for iOS is called Simulator
      (will be referred to as just emulator for the sake of consistency).

### start backend server

```sh
$ rails server
```

### start packager (JS) server (optional)

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

## tips

### Dvorak keyboard layout

by default emulator uses QWERTY as hardware keyboard layout - it
also means that all hotkeys (say, for Developer Menu) use it as well.

it's possible to change it to Dvorak inside emulator:

`Settings` -> `General` -> `Keyboards` -> `Hardware Keyboard` -> `English` -> `Dvorak`

### change screen resolution (scale) in emulator

<https://stackoverflow.com/questions/10481412>

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

### upload file to emulator

just drag any file from Finder onto emulator window - if it's an image it will
be automatically copied to Photos on iOS (I didn't experiment with other types
of files). moreover it will be copied to Photos regardless of what application
is currently opened.

## debugging

<https://facebook.github.io/react-native/docs/debugging.html>

### Developer Menu

`<D-d>`

### device system log (`iOS device syslog`)

```sh
$ react-native log-ios
```

`console.log()` prints to this log.

### reload application in emulator manually

- `<D-d>` -> `Reload`
- `<D-r>`

### enable live/hot reloading

<https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html>

NOTE: hot reloading is recommended over live reload.

- `<D-d>` -> `Enable Live Reload`
- `<D-d>` -> `Enable Hot Reloading`

## troubleshooting

### application build fails in Xcode (duplicate interface definition for class 'RCTView')

<https://github.com/shoutem/ui/issues/134>

Xcode -> `Issue navigator` -> `Buildtime`:

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

solution:

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
  _my_app/Libraries/BVLinearGradient.xcodeproj/BVLinearGradient/_:

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

### application build fails in command line (bundler identifier entry doesn't exist)

<https://github.com/facebook/react-native/issues/7308>

```sh
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

solution:

error was gone after solving problme with `react-native-linear-gradient`
package (see above) and reinstalling all node modules.

### application fails to start (couldn't find preset "es2015")

this is the same error as on Android.

solution:

unlike on Android enabling hot reloading never helped - use the 2nd method to
fix this problem by temporarily removing `babel` section (see Android article).

### application fails to start (no bundle URL present)

<https://github.com/facebook/react-native/issues/12754>

emulator window:

```sh
No bundle URL present.

Make sure you're running a packager server or have included a .jsbundle file
in your application bundle.
```

solution:

it looks like application is run in emulator before packager server is started.

run application again without closing emulator:

```sh
$ react-native run-ios
```

NOTE: this error usually occurs after running application in Android emulator.

### application fails to start (native module cannot be null)

- <https://github.com/zo0r/react-native-push-notification/issues/160>
- <https://github.com/zo0r/react-native-push-notification/issues/279>
- <http://facebook.github.io/react-native/docs/pushnotificationios.html>
- <http://facebook.github.io/react-native/docs/linking-libraries-ios.html>

emulator window, packager server log:

```sh
Native module cannot be null.
```

solution:

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
rnpm-install info Android module react-native-push-notification is already linked
```

solution is to link `PushNotificationIOS` native library manually as instructed in
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html).
