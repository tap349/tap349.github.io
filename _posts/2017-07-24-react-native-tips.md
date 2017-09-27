---
layout: post
title: React Native - Tips
date: 2017-07-24 15:33:55 +0300
access: public
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

## set application icon badge number

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

## dismiss keyboard when tapping outside of TextInput

1. <https://stackoverflow.com/questions/41426862>
2. <https://blog.brainsandbeards.com/react-native-keyboard-handling-tips-a2472b74c114>

by default keyboard is not dismissed when tapping outside of `TextInput`
component - wrap the latter in `ScrollView` component for this to happen:

```javascript
<ScrollView keyboardShouldPersistTaps='handled'>
  <InputSearch
    label='SEARCH'
    value={this.state.search}
    onChangeText={(value) => this.setState({search: value})}
  />
</ScrollView>
```

if there is a `ListView` component somewhere below you'll have to wrap
`ScrollView` component in `View` component - otherwise it's not visible.

## open application in store

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
