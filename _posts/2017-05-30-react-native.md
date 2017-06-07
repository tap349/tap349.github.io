---
layout: post
title: React Native
date: 2017-05-30 15:54:03 +0300
access: public
categories: [react-native]
---

<!-- more -->

## networking

<https://facebook.github.io/react-native/docs/network.html>

RN provides `Fetch API` that allows to fetch content from arbitrary URL:

```javascript
function getUsers() {
  return fetch('https://test.com/users.json')
    .then(response => response.json())
    .then(responseJson => responseJson.users)
    .catch(error => console.error(error))
}
```

`Fetch` methods (as seen above) return promises.

example can be rewritten using `async`/`await` syntax from ES2017:

```javascript
async function getUsers() {
  try {
    let response = await fetch('https://test.com/users.json');
    let responseJson = await response.json();
    return responseJson.users;
  } catch(error) {
    console.error(error)
  }
}
```

## set application icon badge number

<https://stackoverflow.com/questions/40313361>

### Android

not supported natively - use 3rd party libraries like
[ShortcutBadger](https://github.com/leolin310148/ShortcutBadger).

### iOS

supported natively - see
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html)
for instructions how to link native library manually.

### universal solution

use [react-native-push-notification](https://github.com/zo0r/react-native-push-notification)
RN package which:

- uses ShortcutBadger for Android under the hood

  though not all Android launchers are supported (not all launchers support
  showing application icon badger number at all) - e.g. badge number is not
  set on application icon in emulator (Nexus 5X) with the following error in
  device system log (`adb logcat`):

  ```sh
  Caused by: me.leolin.shortcutbadger.ShortcutBadgeException:
    unable to resolve intent: Intent { act=android.intent.action.BADGE_COUNT_UPDATE (has extras) }
  ```

- is supposed to link `PushNotificationIOS` native library for iOS

  the package is supposed to do it automatically but it fails to do so -
  see [iOS troubleshooting]({% post_url 2017-05-25-react-native-ios %})
  (`application fails to start (native module cannot be null)`).
  in a nutshell - link `PushNotificationIOS` library manually.

## linking native libraries

<http://facebook.github.io/react-native/docs/linking-libraries-ios.html>

libraries can be linked:

- automatically

  ```sh
  $ npm install --save <library-with-native-dependencies>
  $ react-native link
  ```

  this method is for npm packages that ship with native libraries.

- manually

  by adding corresponding library (its Xcode project file) to
  your application (see the guide above).

  this method is for independent static libraries that ship with RN -
  they are not included by default so as not to impact binary size.
