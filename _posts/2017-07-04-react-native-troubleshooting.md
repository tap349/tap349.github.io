---
layout: post
title: React Native - Troubleshooting
date: 2017-07-04 18:40:40 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

see [npm - Tips]({% post_url 2017-11-19-npm-tips %}) for `npm_reset`.

Couldn't find preset "es2015"
-----------------------------

emulator window:

```
SyntaxError: TransformError: <app_dir>/node_modules/shallowequal/index.js:
Couldn't find preset "es2015" relative to directory "<app_dir>/node_modules/shallowequal"
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

***UPDATE***

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

React.Children.only expected to receive a single React element child
--------------------------------------------------------------------

device system log:

```
<Critical>: Unhandled JS Exception: React.Children.only expected to receive a single React element child.
```

**solution**

1. <https://facebook.github.io/react-native/docs/touchablehighlight.html>

> TouchableHighlight must have one child (not zero or more than one).
> If you wish to have several child components, wrap them in a View.

errors after upgrading RN to 0.45.1
-----------------------------------

### Cannot find module X

```sh
$ react-native start
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
$ rm -rf node-modules
$ yarn install
$ yarn start
```

### DeviceInfo native module is not installed correctly

emulator window:

```
DeviceInfo native module is not installed correctly
```

**solution**

1. <https://github.com/rebeccahughes/react-native-device-info/issues/176>

rebuild application.

### Unhandled JS Exception: undefined is not an object

emulator window:

```
Unhandled JS Exception: undefined is not an object (evaluating 'PropTypes.shape')
```

**solution**

1. <https://github.com/jsierles/react-native-audio/issues/83>

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

Maximum call stack size exceeded
--------------------------------

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
2. parent component is re-rendered using `forceUpdate()`
  (in my case it's subscribed to store updates)
3. component is re-rendered too
4. action is immediately dispatched again (infinite loop)

to avoid infinite loop dispatch actions and set component state only in

- constructor or
- callbacks that are not immediately invoked when component is being rendered

In next release empty section headers will be rendered
------------------------------------------------------

emulator window:

```
Warning: In next release empty section headers will be rendered. In this
release you can use 'enableEmptySections' flag to render empty section
headers.
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

onEndReached event of ListView keeps on firing
----------------------------------------------

**solution**

1. <https://stackoverflow.com/questions/38531369>
2. <https://github.com/facebook/react-native/issues/6002>
3. <https://facebook.github.io/react-native/docs/refreshcontrol.html>

don't nest `ListView` in `ScrollView` - this is what
causes `onEndReached` event to be triggered again and again.

in most cases it means not to use `ScrollView` at all if you have
to use `ListView`:

- if you used `refreshControl` property of `ScrollView` note
  that `ListView` has the same property
- if you used some header in `ScrollView` it's possible to render
  the very same header in `renderHeader` callback of `ListView`

ListView becomes blank
----------------------

when pushing another page and then going back to page with `ListView`
the latter becomes blank (nothing is rendered where `ListView` is
supposed to be rendered). at the same time adjacent components are
rendered properly (=> it's not that wrapped collection becomes empty).

**solution**

1. <https://github.com/facebook/react-native/issues/8607>

```jsx
<ListView
  removeClippedSubviews={false}
  ...
/>
```

though I guess it's more of a hack than real solution.

SyntaxError wallet.png: Unexpected character (1:0)
--------------------------------------------------

the error occurs sometimes after adding new icon or updating existing one.

**solution**

restart packager (`react-native start`) and reload application in emulator.

errors after upgrading RN to 0.47.0
-----------------------------------

### Module JSTimersExecution is not a registered callable module

1. <https://stackoverflow.com/questions/45594935>

emulator window:

```
Module JSTimersExecution is not a registered callable module (calling callTimers)
```

**solution**

rebuild application.

Actions must be plain objects
-----------------------------

emulator window:

```
Actions must be plain objects. Use custom middleware for async actions.
```

the error occurs when trying to dispatch a thunk (Thunk middleware is applied).

**solution**

I used curly braces instead of parens when defining thunk action creator:

```diff
-export const requestCreateAuthentication = (phone_number) => {
+export const requestCreateAuthentication = (phone_number) => (
  (dispatch, getState, api) => {
    // ...
  }
-}
+)
```

TouchableOpacity ignores initial opacity
----------------------------------------

`TouchableOpacity` ignores `opacity` property -
it's set only when application is hot reloaded in emulator.

**solution**

1. <https://github.com/facebook/react-native/pull/12628>
2. <https://github.com/facebook/react-native/pull/8909>

according to these links the issue has been closed (that is fixed) but
it doesn't look like this (or maybe it's another but related issue).

anyway I've found a workaround - create nested `View` and set
`opacity` property on it instead of `TouchableOpacity` itself:

{% raw %}
```jsx
<TouchableOpacity>
  <View style={{opacity: 0.5}}>
    <Text>{title}</Text>
  </View>
</TouchableOpacity>
```
{% endraw %}

Unknown named module
--------------------

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

ScrollView with unbounded height doesn't grow on scrolling
----------------------------------------------------------

1. <https://stackoverflow.com/a/43525913/3632318>

I use `ScrollView` as a top-level container - when all its content
doesn't fit on the screen (say, on iPhone 4s), the `ScrollView`
doesn't grow when trying to scroll down: hidden content becomes
visible but overflows the container.

the solution is to use `flexGrow: 1` instead of `flex: 1` for container:

{% raw %}
```jsx
<ScrollView contentContainerStyle={{flexGrow: 1}} scrollEnabled={true}>
</ScrollView>
```
{% endraw %}

TextInput inside TouchableOpacity intercepts touches
----------------------------------------------------

when `TouchableOpacity` wraps `TextInput` and the latter is pressed,
`onPress` callback of `TouchableOpacity` is not invoked.

**solution**

1. <https://github.com/facebook/react-native/issues/14958#issuecomment-324237317>

```jsx
<TouchableOpacity onPress={this._handlePress}>
  <View pointerEvents='none'>
    <TextInput editable={false} />
  </View>
</TouchableOpacity>
```

`fontWeight={600}` is not applied to TextInput
-----------------------------------------------

the error occurs if Gill Sans font is used only - setting font
weight works with `Text` and when default font is used instead.

**solution**

TODO

undefined is not an object (evaluating 'Sentry.options.logLevel')
-----------------------------------------------------------------

device system log:

```
<Notice>: { [TypeError: undefined is not an object (evaluating 'Sentry.options.logLevel')] ... }
```

**solution**

<https://github.com/getsentry/react-native-sentry/issues/237#issuecomment-330779566>:

```javascript
Sentry.config(<sentry_endpoint>);
if (!__DEV__) { Sentry.install(); }
```

ScrollView content is partially hidden below when scrolled to the bottom
------------------------------------------------------------------------

make sure that all `ScrollView` parents have `flex: 1`.

ScrollView inside Modal doesn't respect `keyboardShouldPersistTaps='handled'`
-----------------------------------------------------------------------------

1. <https://github.com/facebook/react-native/issues/10138>

make sure to set `keyboardShouldPersistTaps='handled'` on
ALL parent `ScrollView`s outside of `Modal`:

> <Modal> cares about its (scrollview) parent stack,
> I assumed it would be like if rendered on top level

this error seems to be reproduced only when `ScrollView` is
rendered inside `Modal`.

errors after upgrading RN to 0.52.1
-----------------------------------

### Error: While resolving module `react-native-vector-icons/Octicons`

```sh
$ react-native start
...
error: bundling failed: Error: While resolving module `react-native-vector-icons/Octicons`, the Haste package `react-native-vector-icons` was found. However the module `Octicons` could not be found within the package. Indeed, none of these files exist:

  * `<app_dir>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
  * `<app_dir>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons/index(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
```

**solution**

1. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-357405396>
2. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-358018994>

```sh
$ rm ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json
$ react-native start
```

it looks like this file is recreated after each application build so it
makes sense to add this command to `postinstall` script in _package.json_:

```diff
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start",
    "test": "jest",
    "eslint": "eslint",
    "flow": "flow",
+   "postinstall": "rm -f ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json"
  },
```

### fatal error: 'Flurry.h' file not found

```sh
$ react-native run-ios
...
<app_dir>/node_modules/react-native-flurry-analytics/ios/RNFlurryAnalytics.m:2:9: fatal error: 'Flurry.h' file not found
#import <Flurry.h>
        ^~~~~~~~~~
1 error generated.
```

**solution**

reinstall CocoaPods - see instructions in solution for `<PBXGroup ...>
attempted to initialize an object with an unknown UUID` issue.

### CocoaPods could not find compatible versions for pod "react-native-contacts"

```sh
$ pod install
...
[!] CocoaPods could not find compatible versions for pod "react-native-contacts":
  In Podfile:
    react-native-contacts (from `../node_modules/react-native-contacts`)

Specs satisfying the `react-native-contacts (from `../node_modules/react-native-contacts`)` dependency were found, but they required a higher minimum deployment target.
```

**solution**

<https://facebook.github.io/react-native/docs/linking-libraries-ios.html>:

> If your iOS project is using CocoaPods (contains Podfile) and linked library
> has podspec file, then react-native link will link library using Podfile.

pods were added to _ios/Podfile_ after linking corresponding libraries:

```diff
  target 'iceperkapp-tvOSTests' do
    inherit! :search_paths
    # Pods for testing
+   pod 'react-native-contacts', :path => '../node_modules/react-native-contacts'
+
+   pod 'RNDeviceInfo', :path => '../node_modules/react-native-device-info'
+
+   pod 'react-native-image-picker', :path => '../node_modules/react-native-image-picker'
+
+   pod 'BVLinearGradient', :path => '../node_modules/react-native-linear-gradient'
+
+   pod 'react-native-onesignal', :path => '../node_modules/react-native-onesignal'
+
+   pod 'Picker', :path => '../node_modules/react-native-picker'
+
+   pod 'SentryReactNative', :path => '../node_modules/react-native-sentry'
+
+   pod 'RNSVG', :path => '../node_modules/react-native-svg'
+
+   pod 'RNVectorIcons', :path => '../node_modules/react-native-vector-icons'
+
  end
```

remove `iceperkapp-tvOSTests` and `iceperkappTests` targets altogether
and try to install pods again.

### <PBXGroup ...> attempted to initialize an object with an unknown UUID

```sh
$ pod install
...
[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `EEE09AF85CBC4DA8A7C4E137` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.

[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `9DEE1362370047E5817C99E0` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.
...
```

**solution**

1. <https://github.com/CocoaPods/CocoaPods/issues/1822#issuecomment-304815540>

reinstall CocoaPods:

```sh
$ cd ios/
$ pod deintegrate
$ pod install
```

### object file (.../libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)

```sh
$ react-native run-ios
...
ld: warning: object file (<app_dir>/ios/build/Build/Products/Debug-iphonesimulator/libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)
```

**solution**

1. <https://stackoverflow.com/a/32950454/3632318>

error explanation: `react-native-autogrow-textinput` library is
built with `IPHONEOS_DEPLOYMENT_TARGET=9.3` build setting but
the rest of the project (the library is linked to later) has been
built with `IPHONEOS_DEPLOYMENT_TARGET=8.0` build setting =>
hence the error.

the latest version of `react-native-autogrow-textinput` doesn't support
iOS Deployment Target (DT) lower than 9.3 but current DT in iOS project
is 8.0 => change DT in iOS project to 9.3 as well:

| Xcode: `Build Settings` → `Deployment` → `iOS Deployment Target`

### Undefined symbols for architecture x86_64

```sh
$ react-native run-ios
...
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_RCTOneSignal", referenced from:
      objc-class-ref in AppDelegate.o
  "_OBJC_CLASS_$_RNSentry", referenced from:
      objc-class-ref in AppDelegate.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/18#issuecomment-287132994>
2. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-onesignal` and `react-native-sentry` libraries manually
and rebuild application.

### Cannot read property 'func' of undefined

error is shown in device system log and emulator window.

the error occurs when evaluating this line:

```javascript
onPress: React.PropTypes.func.isRequired
```

**solution**

1. <https://stackoverflow.com/a/47878585/3632318>

> PropTypes has been moved to a separate package. Accessing React.PropTypes
> is no longer supported and will be removed completely in React 16.

import `PropTypes` from a separate `prop-types` package.

### Cannot read property 'appVersion' of undefined

error is shown in device system log and emulator window.

the error occurs when evaluating this line in _deviceinfo.js_ file
of `react-native-device-info` package:

```javascript
return RNDeviceInfo.appVersion;
```

**solution**

1. <https://github.com/rebeccahughes/react-native-device-info/issues/52#issuecomment-340212477>
2. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-device-info` library manually and rebuild application.

### Invariant Violation: Native component for "BVLinearGradient" does not exist

error in device system log and emulator window.

**solution**

1. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-linear-gradient` library manually and rebuild application.

### TypeError: Cannot read property 'isPickerShow' of undefined

error in device system log and emulator window.

**solution**

1. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-picker` library manually and rebuild application.

### Cannot read property 'showImagePicker' of undefined

error in device system log and emulator window.

**solution**

1. <https://github.com/react-community/react-native-image-picker/issues/712>

link `react-native-image-picker` library manually and rebuild application.

### TypeError: undefined is not an object (evaluating 'Contacts.getAll')

error in device system log and emulator window.

**solution**

link `react-native-contacts` library manually and rebuild application.

### Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'

```sh
$ cd android && ./gradlew assembleRelease; cd ..
...
sentry-cli update to 1.28.1 is available!
run sentry-cli update to update
error: http error: [55] Failed sending data to the peer (SSL_write() returned SYSCALL, errno = 32)
:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* Where:
Script '<app_dir>/node_modules/react-native-sentry/sentry.gradle' line: 154

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'.
> Process 'command 'node_modules/sentry-cli-binary/bin/sentry-cli'' finished with non-zero exit value 1
```

**solution**

```sh
$ npm update react-native-sentry
$ npm_reset
```

upgrading `react-native-sentry` didn't help - cleaning cache
and reinstalling all node modules fixed the issue.

### production release crashes on startup (in both Android and iOS devices)

**solution**

1. <https://github.com/facebook/react-native/issues/16567#issuecomment-340041357>
2. <https://github.com/facebook/react-native/issues/16542>

<https://github.com/magus/react-native-facebook-login/issues/279>:

> View.propTypes has been removed in 0.49

replace `View.propTypes.style` with `ViewPropTypes.style` in all components.

see also [React Native - Android]({% post_url 2017-05-24-react-native-android %})
(`debugging` section) on how to debug similar problems on Android.

Could not determine java version from '9.0.4'
---------------------------------------------

the error occurs after updating to JDK 9:

```sh
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
Could not determine java version from '9.0.4'.
```

**solution**

1. <https://github.com/facebook/react-native/issues/17688>
2. <https://github.com/facebook/react-native/issues/16536>

probably not the best but the easiest solution is to rollback to JDK 8:

```sh
$ brew cask uninstall java
$ brew tap caskroom/versions
$ brew cask install java8
```

Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
-------------------------------------------------------------

the error occurs when trying to install application with lower version
(`versionCode` in _android/app/build.gradle_) on device that already has
this application installed with higher version:

```sh
$ react-native run-android
...
:app:installDebug FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: com.android.ddmlib.InstallException: Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
```

**solution**

uninstall application on device (emulator) and build application again.

Swiper image (child) is occasionally blank
------------------------------------------

this often happens, say, after removing the child (game in my application) -
adjacent game should become visible but blank area is shown instead.

**solution**

1. <https://github.com/leecade/react-native-swiper/issues/196#issuecomment-249540289>
2. <https://github.com/leecade/react-native-swiper/issues/609>
3. <https://facebook.github.io/react-native/docs/scrollview.html#removeclippedsubviews>

adding `removeClippedSubviews={false}` property to `Swiper` component seems
to solve the problem for the time being (AFAIU when this property is false,
all offscreen children are force rendered - this might be not suitable for
long lists but in my case all lists are short so it shouldn't be a problem).

***UPDATE***

still this doesn't help - the only solution that works thus far is
<https://github.com/leecade/react-native-swiper/issues/609#issuecomment-338190488>.

***UPDATE***

nothing of the above works )

consider using `react-native-snap-carousel` instead of `react-native-swiper`.

TextInput jumps when added dynamically
--------------------------------------

**solution**

set `minHeight` and `initialHeight` properties:

```jsx
<TextInput
  value="foo"
  minHeight={20}
  initialHeight={20}
/>
```

Could not parse the simulator list output
-----------------------------------------

the error occurs after new Xcode has been downloaded (9.3 in my case):

```sh
$ react-native run-ios
Scanning folders for symlinks in /Users/tap/dev/iceperkapp/node_modules (43ms)
Found Xcode workspace iceperkapp.xcworkspace
dyld: Symbol not found: _SimDeviceBootKeyDisabledJobs
  Referenced from: /Applications/Xcode.app/Contents/Developer/usr/bin/simctl
  Expected in: /Library/Developer/PrivateFrameworks/CoreSimulator.framework/Versions/A/CoreSimulator
 in /Applications/Xcode.app/Contents/Developer/usr/bin/simctl

Could not parse the simulator list output
```

**solution**

1. <https://joelennon.com/resolving-could-not-parse-the-simulator-list-output-when-running-react-native-apps-on-ios-simulator>

launch Xcode and install additional required components when prompted.

image uploading doesn't work in simulator
-----------------------------------------

response from AWS:

```xml
<Error>
  <Code>MalformedPOSTRequest</Code>
  <Message>The body of your POST request is not well-formed multipart/form-data.</Message>
</Error>
```

yellow warning in simulator:

```
[RNDebugger] Detected you've enabled Network Inspect and you're using `uri`
in FormData, it will be a problem if you use it for upload, please see the
documentation (https://goo.gl/yEcRrU) for more information.
```

**solution**

1. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/network-inspect-of-chrome-devtools.md>

disable `Network Inspect`:

| Redux DevTools: RMB → `Disable Network Inspect`

Loading dependency graph, done.
-------------------------------

`react-native start` hangs after `Loading dependency graph, done.` message.

**solution**

1. <https://github.com/facebook/react-native/issues/16798>

I'm not sure but maybe reinstalling Watchman helped to resolve the issue:

```sh
$ brew uninstall watchman
$ brew install watchman
```

errors after upgrading RN from 0.52.1 to 0.59.9
-----------------------------------------------

### react-native-push-notification: Appears to be a git repo or submodule.

```
$ npm install
...
npm ERR! path /Users/tap/dev/iceperkapp/node_modules/react-native-push-notification
npm ERR! code EISGIT
npm ERR! git /Users/tap/dev/iceperkapp/node_modules/react-native-push-notification: Appears to be a git repo or submodule.
npm ERR! git     /Users/tap/dev/iceperkapp/node_modules/react-native-push-notification
npm ERR! git Refusing to remove it. Update manually,
npm ERR! git or move it out of the way first.
```

**solution**

1. <https://github.com/zo0r/react-native-push-notification/issues/1057>

```sh
$ rm -rf node_modules/*/.git
```

### incompatible types: ReadableArray cannot be converted to ReadableNativeArray

```sh
$ react-native run-android
...
> Task :react-native-sentry:compileDebugJavaWithJavac FAILED
/Users/tap/dev/iceperkapp/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java:251: error: incompatible types: Read
ableArray cannot be converted to ReadableNativeArray
            addExceptionInterface(eventBuilder, exception.getString("type"), exception.getString("value"), stacktrace.getArray("frames"));
                                                                                                                              ^
Note: /Users/tap/dev/iceperkapp/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java uses or overrides a deprecated
 API.
```

**solution**

```sh
$ npm install --save react-native-sentry@0.43.1
```

NOTE: 0.43.1 is the latest version at the time of writing.

### constructor RNSentryPackage in class RNSentryPackage cannot be applied to given types

```sh
$ react-native run-android
...
> Task :app:compileDebugJavaWithJavac FAILED
/Users/tap/dev/iceperkapp/android/app/src/main/java/com/iceperkapp/MainApplication.java:45: error: constructor RNSentryPackage in class RNSentryPac
kage cannot be applied to given types;
            new RNSentryPackage(MainApplication.this),
            ^
  required: no arguments
  found: MainApplication
  reason: actual and formal argument lists differ in length
1 error
```

**solution**

1. <https://github.com/getsentry/react-native-sentry/issues/490#issuecomment-438680937>

```diff
  // android/app/src/main/java/com/iceperkapp/MainApplication.java

- new RNSentryPackage(MainApplication.this),
+ new RNSentryPackage(),
```

### Cannot fit requested classes in a single dex file

```sh
$ react-native run-android
...
> Task :app:transformDexArchiveWithExternalLibsDexMergerForDebug FAILED
D8: Cannot fit requested classes in a single dex file (# methods: 66744 > 65536)
```

**solution**

1. <https://developer.android.com/studio/build/multidex>

```diff
  // android/app/build.gradle

  defaultConfig {
      applicationId "com.iceperkapp"
      minSdkVersion rootProject.ext.minSdkVersion
      targetSdkVersion rootProject.ext.targetSdkVersion
+     multiDexEnabled true
      versionCode 97
      versionName "3.17"
      // ...
  }
```

### The Google Mobile Ads SDK was initialized incorrectly

emulator:

```
Unfortunately, MyApp has stopped
```

```
$ adb logcat
...
06-08 01:19:30.203  3738  3738 E AndroidRuntime: java.lang.RuntimeException: Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider: java.lang.IllegalStateException:
06-08 01:19:30.203  3738  3738 E AndroidRuntime:
06-08 01:19:30.203  3738  3738 E AndroidRuntime: ******************************************************************************
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * The Google Mobile Ads SDK was initialized incorrectly. AdMob publishers    *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * should follow the instructions here: https://goo.gl/fQ2neu to add a valid  *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * App ID inside the AndroidManifest. Google Ad Manager publishers should     *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: * follow instructions here: https://goo.gl/h17b6x.                           *
06-08 01:19:30.203  3738  3738 E AndroidRuntime: ******************************************************************************
```

**solution**

1. <https://stackoverflow.com/a/53013453/3632318>
2. <https://developers.google.com/admob/android/quick-start#update_your_androidmanifestxml>

```diff
  <!-- android/app/src/main/AndroidManifest.xml -->

  <manifest>
      <application>
+         <meta-data
+             android:name="com.google.android.gms.ads.APPLICATION_ID"
+             android:value="ca-app-pub-9176480316001296~1177893459"/>
      </application>
  </manifest>
```

### Invalid application ID

emulator:

```
Unfortunately, MyApp has stopped
```

```
$ adb logcat
...
06-08 10:37:17.754  4378  4378 E AndroidRuntime: java.lang.RuntimeException: Unable to get provider com.google.android.gms.ads.MobileAdsInitProvider: java.lang.IllegalStateException:
06-08 10:37:17.754  4378  4378 E AndroidRuntime:
06-08 10:37:17.754  4378  4378 E AndroidRuntime: ******************************************************************************
06-08 10:37:17.754  4378  4378 E AndroidRuntime: * Invalid application ID. Follow instructions here: https://goo.gl/fQ2neu to *
06-08 10:37:17.754  4378  4378 E AndroidRuntime: * find your app ID.                                                          *
06-08 10:37:17.754  4378  4378 E AndroidRuntime: ******************************************************************************
```

**solution**

make sure to replace slash with tilde in AdMob App ID:

```diff
  <!-- android/app/src/main/AndroidManifest.xml -->

  <manifest>
      <application>
          <meta-data
+             android:name="com.google.android.gms.ads.APPLICATION_ID"
-             android:value="ca-app-pub-9176480316001296/1177893459"/>
+             android:value="ca-app-pub-9176480316001296~1177893459"/>
      </application>
  </manifest>
```

# Plugin/Preset files are not allowed to export objects, only functions.

```
$ react-native start
...
Loading dependency graph, done.
error: bundling failed: Error: Plugin/Preset files are not allowed to export objects, only functions. In /Users/tap/dev/iceperkapp2/node_modules/babel-preset-flow/lib/index.js
    at createDescriptor (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-descriptors.js:178:11)
    at /Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-descriptors.js:109:50
    at Array.map (<anonymous>)
    at createDescriptors (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-descriptors.js:109:29)
    at createPresetDescriptors (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-descriptors.js:101:10)
    at presets (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-descriptors.js:47:19)
    at mergeChainOpts (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-chain.js:320:26)
    at /Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-chain.js:283:7
    at mergeExtendsChain (/Users/tap/dev/iceperkapp2/node_modules/@babel/core/lib/config/config-chain.js:299:21)
```

**solution**

1. <https://github.com/facebook/flow/issues/7046#issuecomment-443397899>

try updating corresponding npm package:

```
$ npm uninstall babel-preset-flow
$ npm install --save-dev @babel/preset-flow
```

```diff
  // babel.config.js

  module.exports = {
-   presets: ['flow']
+   presets: ['@babel/preset-flow']
  };
```

### The 'decorators' plugin requires a 'decoratorsBeforeExport' option, whose value must be a boolean

```sh
$ react-native start
...
error: bundling failed: Error: The 'decorators' plugin requires a 'decoratorsBeforeExport' option, whose value must be a boolean. If you are migrating from Babylon/Babel 6 or want to use the old decorators proposal, you should use the 'decorators-legacy' plugin instead of 'decorators'.
    at validatePlugins (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/parser/lib/index.js:6093:13)
    at getParser (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/parser/lib/index.js:11286:5)
    at parse (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/parser/lib/index.js:11256:22)
    at parser (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/core/lib/transformation/normalize-file.js:170:34)
    at normalizeFile (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/core/lib/transformation/normalize-file.js:138:11)
    at runSync (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/core/lib/transformation/index.js:44:43)
    at transformSync (/Users/tap/dev/compleader/iceperkapp2/node_modules/@babel/core/lib/transform.js:43:38)
    at Object.transform (/Users/tap/dev/compleader/iceperkapp2/node_modules/metro-react-native-babel-transformer/src/index.js:202:20)
```

**solution**

1. <https://react-redux.js.org/introduction/basic-tutorial#connecting-the-components>

> <https://www.npmjs.com/package/babel-plugin-transform-decorators-legacy#babel--7x>
>
> This plugin is specifically for Babel 6.x. If you're using Babel 7, this plugin
> is not for you. Babel 7's @babel/plugin-proposal-decorators officially supports
> the same logic that this plugin has, but integrates better with Babel 7's other
> plugins.

but after having tried all possible combinations of `decoratorsBeforeExport` and
`legacy` options I still couldn't make new decorators work. so:

> <https://github.com/reduxjs/react-redux/issues/1162#issuecomment-453847059>
>
> We do not recommend using connect as a decorator, because the spec is unstable.

=> it's easier not to use decorators at all (and remove all related packages),
the more especially as it's a recommended approach.

### TypeError: undefined is not an object (evaluating 'props.getItem')

```
$ adb logcat
...
06-10 00:18:39.077 10262 10367 E ReactNativeJS: TypeError: undefined is not an object (evaluating 'props.getItem')
06-10 00:18:39.077 10262 10367 E ReactNativeJS:
06-10 00:18:39.077 10262 10367 E ReactNativeJS: This error is located at:
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in FlatList (at YellowBoxList.js:87)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in RCTView (at View.js:45)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in View (at YellowBoxList.js:79)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in YellowBoxList (at YellowBox.js:104)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in YellowBox (at AppContainer.js:93)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in RCTView (at View.js:45)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in View (at AppContainer.js:115)
06-10 00:18:39.077 10262 10367 E ReactNativeJS:     in AppContainer (at renderApplication.js:34)
06-10 00:18:39.090 10262 10367 E ReactNativeJS: TypeError: undefined is not an object (evaluating 'props.getItem')
```

**solution**

1. <https://github.com/facebook/react-native/issues/21154#issuecomment-439348692>

in this case and in case of many other weird JS errors first try resetting cache:

```
$ react-native start --reset-cache
```
