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

## Couldn't find preset "es2015"

emulator window:

```sh
SyntaxError: TransformError: /Users/tap/dev/my_app/node_modules/shallowequal/index.js:
Couldn't find preset "es2015" relative to directory "/Users/tap/dev/my_app/node_modules/shallowequal"
```

**solution**

it has turned out that `shallowequal` is the only module in _node_modules/_ that
configures babel presets to use in its _package.json_:

```json
  "babel": {
    "presets": [
      "es2015"
    ]
  },
```

quick-and-dirty fix for both Android and iOS:

- remove offending section from _package.json_ of `shallowequal` package
- restart packager service (`npm start`) - no error
- get that section back
- restart packager service (`npm start`) - still no error

even if `shallowequal` package is removed from filesystem and installed again
the error no longer occurs - maybe the 'right' version of `shallowequal` package
is cached somewhere?

NOTE: still this error might occur the next time emulator is run.

[Android] the error might disappear after enabling hot reloading in emulator
(`<D-m>` -> `Enable Hot Reloading`) - enabling live reload has no effect.

[iOS] enabling hot reloading never helped - use the fix above.

all in all IDK why this error occurs and how to fix it in general.

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

emulator window:

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

emulator window:

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

## Maximum call stack size exceeded

emulator window:

```
Maximum call stack size exceeded.
```

device system log:

```
<Warning>: Warning: Cannot update during an existing state transition
(such as within `render` or another component's constructor).
Render methods should be a pure function of props and state;
constructor side-effects are an anti-pattern, but can be moved to
`componentWillMount`.
<Error>: Maximum call stack size exceeded.
<Critical>: Unhandled JS Exception: Maximum call stack size exceeded.
```

**solution**

<https://stackoverflow.com/questions/37387351>

don't dispatch Redux actions in `render()` method or any other methods that
are called when component is being rendered - this will cause an infinite loop:

- action is dispatched when child component is being rendered
- reducers update the store (application state) according to that action
- parent component is re-rendered using `forceUpdate()` since it's
  subscribed to store updates
- child component is rendered again causing action to be dispatched

to avoid infinite loop dispatch actions only in constructor or callbacks
that are not immediately invoked when component is being rendered.
