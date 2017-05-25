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

## troubleshooting

### application build fails in Xcode (duplicate interface definition for class 'RCTView')

xCode -> issue navigator -> Buildtime:

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

solution:

<https://github.com/shoutem/ui/issues/134>

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
  "react-native-linear-gradient": "^2.0.0"
  ```

  ```sh
  $ npm update react-native-linear-gradient
  ```
