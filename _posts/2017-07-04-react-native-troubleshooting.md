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
<hr>

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
- restart packager - no error
- get that section back
- restart packager - still no error

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
$ yarn install
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

DON'T:

- dispatch Redux actions (`this.props.store.dispatch(...)`) or
- set component state (`this.setState(...)`)

in `render()` method or any other method that is called from `render()` method
(that is when component is being rendered) - this will cause an infinite loop.

for example, if you dispatch action when component is being rendered:

1. reducers update the store according to that action
2. parent component is re-rendered using `forceUpdate()`
  (in my case it's subscribed to store updates)
3. component is re-rendered too
4. action is immediately dispatched again (infinite loop)

to avoid infinite loop dispatch actions and set component state only in

- constructor or
- callbacks that are not immediately invoked when component is being rendered

## In next release empty section headers will be rendered

emulator window:

```sh
Warning: In next release empty section headers will be rendered. In this
release you can use 'enableEmptySections' flag to render empty section
headers.
```

**solution**

<https://github.com/FaridSafi/react-native-gifted-listview/issues/39#issuecomment-217073492>

> If you use cloneWithRows then you don't have sections and so there's no issue
> with section headers showing up. The confusing part is that even if you don't
> use sections, it will still throw the warning mentioning sections. In this
> case, you can just set enableEmptySections={true} and forget about it.

```jsx
<ListView
  enableEmptySections={true}
  ...
/>
```

## onEndReached event of ListView keeps on firing

**solution**

- <https://stackoverflow.com/questions/38531369>
- <https://github.com/facebook/react-native/issues/6002>
- <https://facebook.github.io/react-native/docs/refreshcontrol.html>

don't nest `ListView` component in `ScrollView` component -
this is what causes `onEndReached` event to be triggered again and again.

in most cases it means not to use `ScrollView` component at all if
you have to use `ListView` component:

- if you used `refreshControl` property of `ScrollView` component
  note that `ListView` component has the same property
- if you used some header in `ScrollView` component it's possible to render
  the very same header in `renderHeader` callback of `ListView` component
