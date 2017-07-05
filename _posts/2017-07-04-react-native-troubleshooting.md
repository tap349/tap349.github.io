---
layout: post
title: React Native - Troubleshooting
date: 2017-07-04 18:40:40 +0300
access: public
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}

## React.Children.only expected to receive a single React element child

in device system log:

```sh
React.Children.only expected to receive a single React element child.
```

**solution**

<https://facebook.github.io/react-native/docs/touchablehighlight.html>

> TouchableHighlight must have one child (not zero or more than one).
> If you wish to have several child components, wrap them in a View.

## errors after upgrading RN

### Cannot find module X

```sh
$ npm start

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

use yarn instead of npm (somehow it managed to install all required dependencies):

```sh
$ rm package-lock.json
$ rm -rf node-modules/
$ yarn
$ npm start
```

### DeviceInfo native module is not installed correctly

in emulator window:

```sh
DeviceInfo native module is not installed correctly
```

**solution**

<https://github.com/rebeccahughes/react-native-device-info/issues/176>

rebuild application:

```sh
$ react-native run-ios
```

### Unhandled JS Exception: undefined is not an object

in emulator window:

```sh
Unhandled JS Exception: undefined is not an object (evaluating 'PropTypes.shape')
```

**solution**

<https://github.com/jsierles/react-native-audio/issues/83>

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

`react-native-git-upgrade` command downgraded `react` package to
another alpha version and this is what most likely fixed the issue.
