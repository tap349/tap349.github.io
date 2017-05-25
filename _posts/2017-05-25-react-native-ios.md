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

### run application

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
