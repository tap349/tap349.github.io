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

## running

### start server

```sh
$ rails server
```

### run application in emulator

```sh
$ react-native run-ios
```

the first run might take a long time since RN will try to
download and install all required iOS libraries.

## tips

<https://facebook.github.io/react-native/docs/debugging.html>

NOTE: all hotkeys use keys in QWERTY layout!
      that is you have to press `<D-e>` to access Developer Menu
      in case you are using Dvorak.

### console logs

```sh
$ react-native log-ios
```

### Developer Menu

`<D-d>`

### reloading application in emulator manually

- `<D-d>` -> `Reload`
- `<D-r>`

### enable hot reloading

`<D-d>` -> `Enable Hot Reloading`

## troubleshooting

### application build fails in Xcode (duplicate interface definition for class 'RCTView')

<https://github.com/shoutem/ui/issues/134>

xCode -> issue navigator -> Buildtime:

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

solution:

too old version of `react-native-linear-gradient` package was used
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

  Xcode -> Project navigator

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

unlike on Android enabling hot reloading doesn't help here.

still I managed to get rid of this error by making these steps:

- quit emulator
- remove offending section from _package.json_ of `shallowequal` package
- run application in emulator (no error, application is launched)
- quit emulator
- get the section back
- run application again - still no error

even if `shallowequal` package is removed from filesystem and installed again
the error no longer occurs (maybe 'right' version of `shallowequal` package is
cached somewhere?).

all in all IDK why this error occurs and how to fix it in general.

### application fails to start (no bundle URL present)

<https://github.com/facebook/react-native/issues/12754>

error message is displayed in emulator window only:

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
