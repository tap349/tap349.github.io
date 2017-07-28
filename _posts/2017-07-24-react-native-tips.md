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

<https://stackoverflow.com/questions/40313361>

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

<https://stackoverflow.com/questions/41426862>

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

if there is `ListView` component somewhere below you'll have to wrap
`ScrollView` component in `View` component - otherwise it's not visible.
