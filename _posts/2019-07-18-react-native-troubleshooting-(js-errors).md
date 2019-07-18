---
layout: post
title: React Native - Troubleshooting (JS errors)
date: 2019-07-18 16:38:00 +0300
access: public
comments: true
categories: [react-native, js]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## Couldn't find preset "es2015"

emulator window:

```
SyntaxError: TransformError: <APP_DIR>/node_modules/shallowequal/index.js:
Couldn't find preset "es2015" relative to directory "<APP_DIR>/node_modules/shallowequal"
```

**solution**

it has turned out that `shallowequal` is the only module in _node_modules/_ that
configures Babel presets to use in its _package.json_:

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

NOTE: still the error might occur the next time emulator is run.

[Android] the error might disappear after enabling hot reloading in emulator
(`<D-m>` → `Enable Hot Reloading`) - enabling live reload has no effect.

[iOS] enabling hot reloading never helped - use the fix above.

all in all IDK why this error occurs and how to fix it in general.

**_UPDATE_**

1. <https://github.com/dashed/shallowequal/issues/11>
2. <https://github.com/dashed/shallowequal/commit/f515936c8a790fbc225add864265b6c82881c9b1>

bug was fixed in v1.0.2 by moving Babel settings to _.babelrc_ so that they are
not consumed by RN packager by default.

`react-side-effect` is the only package that depends on `shallowequal` package
(according to _package-lock.json_) => update `react-side-effect` to update its
dependencies (including `shallowequal`) to their latest version:

```sh
$ npm update react-side-effect
```

## React.Children.only expected to receive a single React element child

device system log:

```
<Critical>: Unhandled JS Exception: React.Children.only expected to receive a single React element child.
```

**solution**

1. <https://facebook.github.io/react-native/docs/touchablehighlight.html>

> TouchableHighlight must have one child (not zero or more than one). If you
> wish to have several child components, wrap them in a View.

## Maximum call stack size exceeded

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

1. <https://stackoverflow.com/questions/37387351>

DON'T:

- dispatch Redux actions (`this.props.store.dispatch(...)`) or
- set component state (`this.setState(...)`)

in `render()` method or any other method that is called from `render()` method
(that is when component is being rendered) - this will cause an infinite loop.

for example, if you dispatch action when component is being rendered:

1. reducers update the store according to that action
2. parent component is re-rendered using `forceUpdate()` (in my case it's
   subscribed to store updates)
3. component is re-rendered too
4. action is immediately dispatched again (infinite loop)

to avoid infinite loop dispatch actions and set component state only in

- constructor or
- callbacks that are not immediately invoked when component is being rendered

## In next release empty section headers will be rendered

emulator window:

```
Warning: In next release empty section headers will be rendered. In this
release you can use 'enableEmptySections' flag to render empty section headers.
```

**solution**

1. <https://github.com/FaridSafi/react-native-gifted-listview/issues/39#issuecomment-217073492>

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

1. <https://stackoverflow.com/questions/38531369>
2. <https://github.com/facebook/react-native/issues/6002>
3. <https://facebook.github.io/react-native/docs/refreshcontrol.html>

don't nest `ListView` in `ScrollView` - this is what causes `onEndReached` event
to be triggered again and again.

in most cases it means not to use `ScrollView` at all if you have to use
`ListView`:

- if you use `refreshControl` property of `ScrollView` note that `ListView` has
  the same property
- if you use some header in `ScrollView` it's possible to render the very same
  header in `renderHeader` callback of `ListView`

## ListView becomes blank

when pushing another page and then going back to page with `ListView` the latter
becomes blank (nothing is rendered where `ListView` is supposed to be rendered).
at the same time adjacent components are rendered properly (=> it's not that
wrapped collection becomes empty).

**solution**

1. <https://github.com/facebook/react-native/issues/8607>

```jsx
<ListView
  removeClippedSubviews={false}
  ...
/>
```

though I guess it's more of a hack than real solution.

## SyntaxError wallet.png: Unexpected character (1:0)

the error occurs sometimes after adding new icon or updating existing one.

**solution**

restart packager (`react-native start`) and reload application in emulator.

## Actions must be plain objects

emulator window:

```
Actions must be plain objects. Use custom middleware for async actions.
```

the error occurs when trying to dispatch a thunk (Thunk middleware is applied).

**solution**

I used curly braces instead of parens when defining thunk action creator:

```diff
- export const requestCreateAuthentication = (phone_number) => {
+ export const requestCreateAuthentication = (phone_number) => (
   (dispatch, getState, api) => {
     // ...
   }
- }
+ )
```

## TouchableOpacity ignores initial opacity

`TouchableOpacity` ignores `opacity` property - it's set only when application
is hot reloaded in emulator.

**solution**

1. <https://github.com/facebook/react-native/pull/12628>
2. <https://github.com/facebook/react-native/pull/8909>

according to these links the issue has been closed (that is fixed) but it
doesn't look like this (or maybe it's another but related issue).

anyway I've found a workaround - create nested `View` and set `opacity` property
on it instead of `TouchableOpacity` itself:

{% raw %}

```jsx
<TouchableOpacity>
  <View style={{opacity: 0.5}}>
    <Text>{title}</Text>
  </View>
</TouchableOpacity>
```

{% endraw %}

## Unknown named module

the error occurs when trying to load image using `Image`.

device system log:

```
<Notice>: { [Error: Unknown named module: '~/components/_new/graphics/images/ion-ios-person.png'] ... }
```

**solution**

1. <https://github.com/facebook/react-native/issues/2481>

it turns out you cannot load images dynamically - provide static URI instead
(that is don't construct it dynamically, say, using variable interpolation):

{% raw %}

```jsx
<Image
  style={{height: 30, width: 35}}
  source={require('~/components/_new/graphics/images/ion-person-stalker.png')}
/>
```

{% endraw %}

## ScrollView with unbounded height doesn't grow on scrolling

1. <https://stackoverflow.com/a/43525913/3632318>

I use `ScrollView` as a top-level container - when all its content doesn't fit
on the screen (say, on iPhone 4s), the `ScrollView` doesn't grow when trying to
scroll down: hidden content becomes visible but overflows the container.

the solution is to use `flexGrow: 1` instead of `flex: 1` for container:

{% raw %}

```jsx
<ScrollView
  contentContainerStyle={{flexGrow: 1}}
  scrollEnabled={true}
></ScrollView>
```

{% endraw %}

## TextInput inside TouchableOpacity intercepts touches

when `TouchableOpacity` wraps `TextInput` and the latter is pressed, `onPress`
callback of `TouchableOpacity` is not invoked.

**solution**

1. <https://github.com/facebook/react-native/issues/14958#issuecomment-324237317>

```jsx
<TouchableOpacity onPress={this._handlePress}>
  <View pointerEvents='none'>
    <TextInput editable={false} />
  </View>
</TouchableOpacity>
```

## `fontWeight={600}` is not applied to TextInput

the error occurs if Gill Sans font is used only - setting font weight works with
`Text` and when default font is used instead.

**solution**

TODO

## undefined is not an object (evaluating 'Sentry.options.logLevel')

device system log:

```
<Notice>: { [TypeError: undefined is not an object (evaluating 'Sentry.options.logLevel')] ... }
```

**solution**

<https://github.com/getsentry/react-native-sentry/issues/237#issuecomment-330779566>:

```javascript
Sentry.config(SENTRY_ENDPOINT);
if (!__DEV__) {
  Sentry.install();
}
```

## ScrollView content is partially hidden below when scrolled to the bottom

make sure that all `ScrollView` parents have `flex: 1`.

## ScrollView inside Modal doesn't respect `keyboardShouldPersistTaps='handled'`

1. <https://github.com/facebook/react-native/issues/10138>

make sure to set `keyboardShouldPersistTaps='handled'` on ALL parent
`ScrollView`s outside of `Modal`:

> <Modal> cares about its (scrollview) parent stack, I assumed it would be like
> if rendered on top level

this error seems to be reproduced only when `ScrollView` is rendered inside
`Modal`.

## Swiper image (child) is occasionally blank

this often happens, say, after removing the child (game in my application) -
adjacent game should become visible but blank area is shown instead.

**solution**

1. <https://github.com/leecade/react-native-swiper/issues/196#issuecomment-249540289>
2. <https://github.com/leecade/react-native-swiper/issues/609>
3. <https://facebook.github.io/react-native/docs/scrollview.html#removeclippedsubviews>

adding `removeClippedSubviews={false}` property to `Swiper` component seems to
solve the problem for the time being (AFAIU when this property is false, all
offscreen children are force rendered - this might be not suitable for long
lists but in my case all lists are short so it shouldn't be a problem).

**_UPDATE_**

still this doesn't help - the only solution that works thus far is
<https://github.com/leecade/react-native-swiper/issues/609#issuecomment-338190488>.

**_UPDATE_**

nothing of the above works )

consider using `react-native-snap-carousel` instead of `react-native-swiper`.

## TextInput jumps when added dynamically

**solution**

set `minHeight` and `initialHeight` properties:

```jsx
<TextInput value='foo' minHeight={20} initialHeight={20} />
```

## image uploading doesn't work in emulator

response from AWS:

```xml
<Error>
  <Code>MalformedPOSTRequest</Code>
  <Message>The body of your POST request is not well-formed multipart/form-data.</Message>
</Error>
```

emulator window:

```
[RNDebugger] Detected you've enabled Network Inspect and you're using `uri`
in FormData, it will be a problem if you use it for upload, please see the
documentation (https://goo.gl/yEcRrU) for more information.
```

**solution**

1. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/network-inspect-of-chrome-devtools.md>

disable `Network Inspect`:

<!-- prettier-ignore -->
| Redux DevTools: RMB → `Disable Network Inspect`

## Loading dependency graph, done.

`react-native start` hangs after `Loading dependency graph, done.` message.

**solution**

1. <https://github.com/facebook/react-native/issues/16798>

I'm not sure but maybe reinstalling Watchman helped to resolve the issue:

```sh
$ brew uninstall watchman
$ brew install watchman
```
