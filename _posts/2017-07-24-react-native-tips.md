---
layout: post
title: React Native - Tips
date: 2017-07-24 15:33:55 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) set application icon badge number
------------------------------------------

1. <https://stackoverflow.com/questions/40313361>

### Android

not supported natively - use 3rd party libraries like
[ShortcutBadger](https://github.com/leolin310148/ShortcutBadger).

though not all Android launchers are supported (not all launchers support
showing application icon badger number at all) - e.g. badge number is not
set on application icon in emulator (Nexus 5X) with the following error in
device system log (`adb logcat`):

```sh
Caused by: me.leolin.shortcutbadger.ShortcutBadgeException:
  unable to resolve intent: Intent { act=android.intent.action.BADGE_COUNT_UPDATE (has extras) }
```

### iOS

supported natively - see
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html)
for instructions how to link `PushNotificationIOS` native library manually.

**universal solution**

use [react-native-push-notification](https://github.com/zo0r/react-native-push-notification)
RN package which:

- uses `ShortcutBadger` under the hood for Android
- links `PushNotificationIOS` native library for iOS

  the package is supposed to do it automatically but it fails to do so -
  see [iOS troubleshooting]({% post_url 2017-05-25-react-native-ios %})
  (`application fails to start (native module cannot be null)`).

  in a nutshell - link `PushNotificationIOS` native library manually.

(how to) dismiss keyboard when tapping outside of TextInput
-----------------------------------------------------------

1. <https://stackoverflow.com/questions/41426862>
2. <https://blog.brainsandbeards.com/react-native-keyboard-handling-tips-a2472b74c114>

by default keyboard is not dismissed when tapping outside of
`TextInput` - wrap the latter in `ScrollView` for this to happen:

```jsx
<ScrollView keyboardShouldPersistTaps='handled'>
  <InputSearch
    label='SEARCH'
    value={this.state.search}
    onChangeText={(value) => this.setState({search: value})}
  />
</ScrollView>
```

if there is a `ListView` somewhere below you'll have to wrap
`ScrollView` in `View` - otherwise it's not visible.

(how to) dismiss keyboard when pressing `<CR>` on keyboard
----------------------------------------------------------

1. <https://facebook.github.io/react-native/docs/textinput.html>

```jsx
<TextInput
  blurOnSubmit={true}
  ...
/>
```

(how to) open application in store
----------------------------------

1. <https://stackoverflow.com/questions/35612383>

first find out application IDs in App Store and Play Store:

- iOS: <https://linkmaker.itunes.apple.com>
- Android: <https://play.google.com/store/apps>

examples of application IDs:

- iOS: `id1111111111`
- Android: `com.myapp`

examples of final URLs:

- iOS: `itms://itunes.apple.com/ru/app/id1111111111?mt=8`
- Android: `market://details?id=com.myapp`

<http://facebook.github.io/react-native/docs/linking.html#canopenurl>:

since iOS 9 it's necessary to allow to query schemas explicitly in
_Info.plist_ or else `canOpenUrl` will always return `false`.

install `react-native-app-link` package to handle it for you:

```sh
$ yarn add react-native-app-link --save
```

NOTE: still opening application in App Store doesn't work in emulator.

process invalid props for all mounted components
------------------------------------------------

I had one component (`TeamInfoPage`) pushed on top of another one
(`TeamPage`) using navigator - the 1st component is not unmounted
in this case.

it has turned out that `TeamPage` component is still updated in
the background.

so make sure to allow for invalid properties in hidden component
caused by some action in pushed component (in my case team was
destroyed in `TeamInfoPage` and I had to process undefined team
in `TeamPage` component).

don't return null from top level component's `render` method
------------------------------------------------------------

in my case returning null from top level component (i.e. component
rendered right in _App.js_ for specified route) caused application
to crash.

**UPDATE**

well, I cannot reproduce it now - returning null just hides component.
according to docs returning null used to be not supported in the past
but that is no longer the case.

so just be cautious when returning null - if it breaks anything, return,
say, `Text` instead.

log error messages in `catch` clauses
-------------------------------------

in case of unhandled promise rejection there might be no relevant
messages in `react-native log-ios` output - make sure that errors
are logged in corresponding `catch` clauses in that case.

(how to) to use absolute paths for imports
------------------------------------------

1. <https://medium.com/@davidjwoody/6b06ae3f65d1>

the problem is that it's necessary to include application
directory name if you want to use absolute paths:

```javascript
import MyComponent from '<app_dir>/app/components/MyComponent';
```

it's possible to use `babel-plugin-root-import` plugin but Vim's
`gf` command (opens file under the cursor) doesn't understand its
notation (`~` stands for `<app_dir>/app` by default).

the solution is to create _package.json_ in directory from which
you want to import (it's _app/_ as a rule):

```json
{
  "name": "app"
}
```

this name can then be used when importing modules:

```javascript
import MyComponent from 'app/components/MyComponent';
```

in fact the name can be anything (say, `app1`) but for Vim to find
the file being imported it must match application top-level directory
name (`app`) - that's the whole point of using custom name instead of
`~` prefix.

behaviour of nested ScrollViews
-------------------------------

`ScrollView` is scrollable if `scrollEnabled={true}` (default value).

```
ScrollView 1 (scrollable)
  ScrollView 2 (not scrollable)
    ScrollView 3 (scrollable)
```

=> `ScrollView 3` scrolls inside of `ScrollView 2`.

```
ScrollView 1 (scrollable)
  ScrollView 2 (scrollable)
    ScrollView 3 (scrollable)
```

=> `ScrollView 3` doesn’t scroll inside of `ScrollView 2`
but they scroll together inside `ScrollView 1`.

but in general it's better to avoid nested `ScrollView`s.

(how to) add comments inside JSX
--------------------------------

```jsx
<MyComponent>
  {/* some comment... */}
  {42}
</MyComponent>
```

(how to) add newline inside Text
---------------------------------

```jsx
<Text>foo{'\n'}bar</Text>
```

(how to) add whitespace (not NBSP) inside Text
----------------------------------------------

```jsx
<Text>
  click{' '}
  <Text>
    "this link"
  </Text>
</Text>
```

(how to) make animations smooth
-------------------------------

1. <https://facebook.github.io/react-native/docs/interactionmanager.html>

fetching data in the background might prevent animations from running smoothly
so schedule fetching data after all interactions have completed (animations are
interactions as well):

```javascript
import {
  InteractionManager,
} from 'react-native';

export default class ShowPage extends React.Component {
  state = {
    interactionsComplete: false,
  }

  componentDidMount () {
    InteractionManager.runAfterInteractions(() => {
      Promise
        .all([
          this.props.getGame(),
          this.props.getGameStat(),
        ])
        .then(() => this.setState({interactionsComplete: true}))
        .catch(AlertHelpers.serverError);
    });
  }

  render () {
    if (!this.state.interactionsComplete) { return <IndicatorHOC />; }
    // ...
  }
}
```

(how to) disable keyboard for not editable TextInput
----------------------------------------------------

on Android keyboard pops up when not editable `TextInput` is tapped.

**solution**

in my case this was caused by focusing `TextInput` on each tap manually
(it's wrapped into `TouchableWithoutFeedback` to extend tappable area):

```jsx
render () {
  return (
    <TouchableWithoutFeedback onPress={this._handlePress}>
      <View style={this._containerStyle()}>
        {this._renderTextInput()}
        {this._renderActivityIndicactor()}
        {this.props.children}
      </View>
    </TouchableWithoutFeedback>
  );
}

// this._textInput is a ref to TextInput
_handlePress = () => {
  this._textInput.focus();
}
```

solution is to disable focusing for not editable `TextInput`:

```diff
  _handlePress = () => {
+   if (!this.props.editable) { return; }
    this._textInput.focus();
  }
```
