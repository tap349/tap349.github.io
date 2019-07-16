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

see [npm/Yarn - Tips]({% post_url 2017-11-19-npm-yarn-tips %}) for `npm_reset`.

try resetting cache first before googling the error:

```
$ react-native start --reset-cache
```

JS errors
---------

### Couldn't find preset "es2015"

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

### React.Children.only expected to receive a single React element child

device system log:

```
<Critical>: Unhandled JS Exception: React.Children.only expected to receive a single React element child.
```

**solution**

1. <https://facebook.github.io/react-native/docs/touchablehighlight.html>

> TouchableHighlight must have one child (not zero or more than one).
> If you wish to have several child components, wrap them in a View.

### Maximum call stack size exceeded

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

### In next release empty section headers will be rendered

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

### onEndReached event of ListView keeps on firing

**solution**

1. <https://stackoverflow.com/questions/38531369>
2. <https://github.com/facebook/react-native/issues/6002>
3. <https://facebook.github.io/react-native/docs/refreshcontrol.html>

don't nest `ListView` in `ScrollView` - this is what causes `onEndReached`
event to be triggered again and again.

in most cases it means not to use `ScrollView` at all if you have to use
`ListView`:

- if you use `refreshControl` property of `ScrollView` note that `ListView` has
  the same property
- if you use some header in `ScrollView` it's possible to render the very same
  header in `renderHeader` callback of `ListView`

### ListView becomes blank

when pushing another page and then going back to page with `ListView` the
latter becomes blank (nothing is rendered where `ListView` is supposed to
be rendered). at the same time adjacent components are rendered properly
(=> it's not that wrapped collection becomes empty).

**solution**

1. <https://github.com/facebook/react-native/issues/8607>

```jsx
<ListView
  removeClippedSubviews={false}
  ...
/>
```

though I guess it's more of a hack than real solution.

### SyntaxError wallet.png: Unexpected character (1:0)

the error occurs sometimes after adding new icon or updating existing one.

**solution**

restart packager (`react-native start`) and reload application in emulator.

### Actions must be plain objects

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

### TouchableOpacity ignores initial opacity

`TouchableOpacity` ignores `opacity` property - it's set only when application
is hot reloaded in emulator.

**solution**

1. <https://github.com/facebook/react-native/pull/12628>
2. <https://github.com/facebook/react-native/pull/8909>

according to these links the issue has been closed (that is fixed) but it
doesn't look like this (or maybe it's another but related issue).

anyway I've found a workaround - create nested `View` and set `opacity`
property on it instead of `TouchableOpacity` itself:

{% raw %}
```jsx
<TouchableOpacity>
  <View style={{opacity: 0.5}}>
    <Text>{title}</Text>
  </View>
</TouchableOpacity>
```
{% endraw %}

### Unknown named module

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

### ScrollView with unbounded height doesn't grow on scrolling

1. <https://stackoverflow.com/a/43525913/3632318>

I use `ScrollView` as a top-level container - when all its content doesn't fit
on the screen (say, on iPhone 4s), the `ScrollView` doesn't grow when trying to
scroll down: hidden content becomes visible but overflows the container.

the solution is to use `flexGrow: 1` instead of `flex: 1` for container:

{% raw %}
```jsx
<ScrollView contentContainerStyle={{flexGrow: 1}} scrollEnabled={true}>
</ScrollView>
```
{% endraw %}

### TextInput inside TouchableOpacity intercepts touches

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

### `fontWeight={600}` is not applied to TextInput

the error occurs if Gill Sans font is used only - setting font weight works with
`Text` and when default font is used instead.

**solution**

TODO

### undefined is not an object (evaluating 'Sentry.options.logLevel')

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

### ScrollView content is partially hidden below when scrolled to the bottom

make sure that all `ScrollView` parents have `flex: 1`.

### ScrollView inside Modal doesn't respect `keyboardShouldPersistTaps='handled'`

1. <https://github.com/facebook/react-native/issues/10138>

make sure to set `keyboardShouldPersistTaps='handled'` on ALL parent
`ScrollView`s outside of `Modal`:

> <Modal> cares about its (scrollview) parent stack, I assumed it would be like
> if rendered on top level

this error seems to be reproduced only when `ScrollView` is rendered inside
`Modal`.

### Swiper image (child) is occasionally blank

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

### TextInput jumps when added dynamically

**solution**

set `minHeight` and `initialHeight` properties:

```jsx
<TextInput
  value="foo"
  minHeight={20}
  initialHeight={20}
/>
```

### image uploading doesn't work in emulator

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

| Redux DevTools: RMB → `Disable Network Inspect`

### Loading dependency graph, done.

`react-native start` hangs after `Loading dependency graph, done.` message.

**solution**

1. <https://github.com/facebook/react-native/issues/16798>

I'm not sure but maybe reinstalling Watchman helped to resolve the issue:

```sh
$ brew uninstall watchman
$ brew install watchman
```

iOS errors
----------

### Could not parse the simulator list output

the error occurs after new Xcode has been downloaded (9.3 in my case):

```
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (43ms)
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

### duplicate interface definition for class 'RCTView'

1. <https://github.com/shoutem/ui/issues/134>

| Xcode: `Issue navigator` → `Buildtime`

```sh
> BVLinearGradient:
  > Semantic issue
    > Duplicate interface definition for class 'RCTView'
    > Property has a previous declaration
    ...
```

**solution**

outdated version of `react-native-linear-gradient` package was used (it provides
`LinearGradient` component).

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

that is currently used version of `react-native-linear-gradient` package is not
even supposed to support this version of react-native.

there are 2 ways to solve the problem:

- manually edit files in `react-native-linear-gradient` package

  | Xcode: `Project navigator`

  replace old imports with new ones in all files inside
  _\<MY\_APP>/Libraries/BVLinearGradient.xcodeproj/BVLinearGradient/_:

  ```objc
  // old import
  //#import "RCTView.h"
  // new import
  #import <React/RCTView.h>
  ```

- update `react-native-linear-gradient` package

  _package.json_:

  ```json
  "dependencies": {
    ...
    "react-native-linear-gradient": "^2.0.0",
    ...
  },
  ```

  ```sh
  $ npm update react-native-linear-gradient
  ```

### Entry, ":CFBundleIdentifier", Does Not Exist

1. <https://github.com/facebook/react-native/issues/7308>

```sh
$ react-native run-ios
...
Print: Entry, ":CFBundleIdentifier", Does Not Exist
```

**solution**

error was gone after solving problem with `react-native-linear-gradient` package
(see above) and reinstalling all node modules.

### No bundle URL present

1. <https://github.com/facebook/react-native/issues/12754>

emulator window:

```
No bundle URL present.

Make sure you're running a packager or have included a .jsbundle file in your
application bundle.
```

**solution**

it looks like application is run in emulator before packager is started.

run application again without closing emulator:

```sh
$ react-native run-ios
```

NOTE: the error usually occurs after running application in Android emulator.

### Native module cannot be null

1. <https://github.com/zo0r/react-native-push-notification/issues/160>
2. <https://github.com/zo0r/react-native-push-notification/issues/279>
3. <http://facebook.github.io/react-native/docs/pushnotificationios.html>
4. <http://facebook.github.io/react-native/docs/linking-libraries-ios.html>

packager log:

```sh
Native module cannot be null.
```

**solution**

the error occurred after installing `react-native-push-notification` package,
linking its native dependencies and trying to launch application:

```sh
$ npm install --save react-native-push-notification
$ react-native link react-native-push-notification
$ react-native run-ios
```

but it has turned out `PushNotificationIOS` library has not been linked to my
iOS project automatically (note there's no message about linking `iOS module`):

```sh
$ react-native link react-native-push-notification
Scanning 587 folders for symlinks in <APP_DIR>/node_modules (6ms)
npm-install info Android module react-native-push-notification is already linked
```

solution is to link `PushNotificationIOS` library manually as instructed in
[PushNotificationIOS](http://facebook.github.io/react-native/docs/pushnotificationios.html).

### excessive logging in device system log

that is when running `react-native log-ios`.

**solution**

1. <https://stackoverflow.com/questions/37800790>
2. <https://github.com/bradmartin/nativescript-videoplayer/issues/76>

setting `OS_ACTIVITY_MODE` and `OS_ACTIVITY_DT_MODE` environment variables in
Xcode project had no effect (<https://stackoverflow.com/questions/37800790>).

the only thing that helped to reduce the amount of logging output was exporting
`SIMCTL_CHILD_OS_ACTIVITY_MODE` environment variable in the shell before starting
application (<https://github.com/bradmartin/nativescript-videoplayer/issues/76>):

```sh
$ SIMCTL_CHILD_OS_ACTIVITY_MODE="disable" react-native run-ios
```

or else this variable can be exported for all shells in _~/.zshenv_:

```zsh
export SIMCTL_CHILD_OS_ACTIVITY_MODE="disable"
```

### Domain: NSURLErrorDomain, Error Code: -1200

`WebView` component's content when trying to open HTTPS URL:

```
Error loading page

Domain: NSURLErrorDomain
Error Code: -1200
Description: An SSL error has occurred and a secure connection to the server cannot be made
```

device system log:

```
<Warning>: NSURLSession/NSURLConnection HTTP load failed (kCFStreamErrorDomainSSL, -9824)
<Warning>: 'Encountered an error loading page', { domain: 'NSURLErrorDomain',
  loading: false,
  description: 'An SSL error has occurred and a secure connection to the server cannot be made.',
  target: 398,
  code: -1200,
  canGoBack: false,
  title: '',
  url: '',
  canGoForward: false }
```

**solution**

1. <https://stackoverflow.com/a/30882037/3632318>
2. <https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/>
3. <https://github.com/vinhnx/iOS-notes/issues/1>

problem was solved by disabling forward secrecy support
for offending URL in _Info.plist_:

```diff
  ...
  <key>NSAppTransportSecurity</key>
  <dict>
    <key>NSExceptionDomains</key>
    <dict>
      <key>localhost</key>
      <dict>
        <key>NSExceptionAllowsInsecureHTTPLoads</key>
        <true/>
      </dict>
+     <key>acs.***.ru</key>
+     <dict>
+       <key>NSTemporaryExceptionRequiresForwardSecrecy</key>
+       <false/>
+     </dict>
    </dict>
  </dict>
  ...
```

or else edit _Info.plist_ in Xcode:

| Xcode: `Project navigator` → _\<MY\_APP>/\<MY\_APP>/Info.plist_

### Library not loaded: .../CoreSimulator

the error occurred after updating Xcode from 8.2.1 to 9.2.

```sh
$ react-native run-ios
...
instruments[2536:17017] [MT] DVTPlugInLoading: Failed to load code for plug-in com.apple.xray.discovery.mobiledevice (/Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin), error = Error Domain=NSCocoaErrorDomain Code=3587 "dlopen_preflight(/Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/MacOS/XRMobileDeviceDiscoveryPlugIn): Library not loaded: /Library/Developer/PrivateFrameworks/CoreSimulator.framework/Versions/A/CoreSimulator
  Referenced from: /Applications/Xcode.app/Contents/Applications/Instruments.app/Contents/PlugIns/XRMobileDeviceDiscoveryPlugIn.xrplugin/Contents/MacOS/XRMobileDeviceDiscoveryPlugIn
  Reason: image not found"
...
```

**solution**

1. <https://github.com/facebook/react-native/issues/14342>

install additional components - you'll be prompted to install these components
when you first start Xcode after updating it.

### ld: library not found for -lFlurry-iOS-SDK

build inside Xcode failed:

```
ld: warning: directory not found for option '-L/Users/tap/Library/Developer/Xcode/DerivedData/iceperkapp-agmwzlgvaynpvuadwqakyphzpfof/Build/Products/Debug-iphoneos/Flurry-iOS-SDK'
ld: library not found for -lFlurry-iOS-SDK
```

**solution**

> <https://github.com/xxsnakerxx/react-native-flurry-analytics/issues/8>
>
> Try closing the Xcode project and instead open <project name>.xcworkspace.
> Since this is using Cocopods you need to use the workspace instead for
> everything to get resolved correctly.

see also the tip `(how to) build application in Xcode` on why to open
workspace rather than a project when building application in Xcode.

still library might be really missing - in case you haven't linked it or
automatic linking by `react-native link` failed.

see the tip `(how to) link library with native dependencies manually` on
how to link library manually in the latter case.

### <PBXGroup ...> attempted to initialize an object with an unknown UUID

```
$ cd ios
$ pod install
...
[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `EEE09AF85CBC4DA8A7C4E137` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.

[!] `<PBXGroup name=`Recovered References` UUID=`54040AF61FBD99E400048638`>` attempted to initialize an object with an unknown UUID. `9DEE1362370047E5817C99E0` for attribute: `children`. This can be the result of a merge and  the unknown UUID is being discarded.
...
```

**solution**

1. <https://github.com/CocoaPods/CocoaPods/issues/1822#issuecomment-304815540>

these warnings might be gone after running `pod install` command because it
updates _ios/iceperkapp.xcodeproj/project.pbxproj_ fixing or removing unknown
UUIDs along the way:

```sh
$ cd ios
$ pod install
```

see the tip on how to repair CocoaPods if it doesn't help.

### CocoaPods could not find compatible versions for pod "react-native-contacts"

```sh
$ cd ios
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

remove `iceperkapp-tvOSTests` and `iceperkappTests` targets from _ios/Podfile_.

### CocoaPods could not find compatible versions for pod "Google-Mobile-Ads-SDK"

```sh
$ cd ios
$ pod install
...
Analyzing dependencies
[!] CocoaPods could not find compatible versions for pod "Google-Mobile-Ads-SDK":
  In snapshot (Podfile.lock):
    Google-Mobile-Ads-SDK (= 7.29.0)

  In Podfile:
    Google-Mobile-Ads-SDK

None of your spec sources contain a spec satisfying the dependencies: `Google-Mobile-Ads-SDK, Google-Mobile-Ads-SDK (= 7.29.0)`.

You have either:
 * out-of-date source repos which you can update with `pod repo update` or with `pod install --repo-update`.
 * mistyped the name or version.
 * not added the source repo that hosts the Podspec to your Podfile.

Note: as of CocoaPods 1.0, `pod repo update` does not happen on `pod install` by default.
```

**solution**

```sh
$ pod repo update
```

### CocoaPods could not find compatible versions for pod "RNCAsyncStorage"

```sh
$ cd ios
$ pod install
...
Fetching podspec for `RNCAsyncStorage` from `../node_modules/@react-native-community/async-storage`
[!] CocoaPods could not find compatible versions for pod "RNCAsyncStorage":
  In Podfile:
    RNCAsyncStorage (from `../node_modules/@react-native-community/async-storage`)

Specs satisfying the `RNCAsyncStorage (from `../node_modules/@react-native-community/async-storage`)` dependency were found, but they required a higher minimum deployment target.
```

**solution**

find minimum deployment target for `RNCAsyncStorage` pod:

```ruby
# node_modules/@react-native-community/async-storage/RNCAsyncStorage.podspec

Pod::Spec.new do |s|
  # ...
  s.platform     = :ios, "9.0"
end
```

use it as a global platform for your project in _ios/Podfile_:

```diff
  # ios/Podfile

- platform :ios, '8.0'
+ platform :ios, '9.0'
```

### Could not find iPhone 6 simulator

```sh
$ react-native run-ios
Scanning folders for symlinks in <APP_DIR>/node_modules (13ms)
Found Xcode workspace iceperkapp.xcworkspace

Could not find iPhone 6 simulator
```

**solution**

1. <https://github.com/facebook/react-native/issues/23282>

upgrade RN.

### Undefined symbol: \_OBJC_METACLASS_$_RCTEventEmitter

1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-add-notification-service-extension>

build inside Xcode failed:

```
Undefined symbols for architecture arm64:
  "_OBJC_METACLASS_$_RCTEventEmitter", referenced from:
      _OBJC_METACLASS_$_RCTOneSignalEventEmitter in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_OBJC_CLASS_$_RCTEventEmitter", referenced from:
      _OBJC_CLASS_$_RCTOneSignalEventEmitter in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTRegisterModule", referenced from:
      +[RCTOneSignalEventEmitter load] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTRunningInAppExtension", referenced from:
      -[RCTOneSignalEventEmitter checkPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter requestPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter getPermissionSubscriptionState:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_RCTSharedApplication", referenced from:
      -[RCTOneSignalEventEmitter checkPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
      -[RCTOneSignalEventEmitter requestPermissions:] in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
  "_OBJC_CLASS_$_RCTConvert", referenced from:
      objc-class-ref in libRCTOneSignal.a(RCTOneSignalEventEmitter.o)
ld: symbol(s) not found for architecture arm64
```

error occurred after adding notification service extension to Xcode project
(see the link).

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/368#issuecomment-441473734>

- install React Native SDK without Cocoapods

  1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-installing-with-cocoapods-ios>

  ```diff
    # ios/Podfile

    target 'iceperkapp' do
      # ...
  -   pod 'react-native-onesignal', path: '../node_modules/react-native-onesignal'
    end
  ```

- install Notification Service Extension without Cocoapods

  1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-without-cocoapods>

### Invalid bitcode version (Producer: '802.0.41.0_0' Reader: '800.0.42.1_0')

| Xcode: `Product` (top menu) → `Archive`

```
error: Invalid bitcode version (Producer: '802.0.42.0_0' Reader: '800.0.42.1_0')
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://stackoverflow.com/questions/43480556/xcode-8-2-1-error-invalid-bitcode-version-producer-802-0-41-0-0-reader>

either:

- *[RECOMMENDED]* update Xcode to 8.3
- disable bitcode in `Build Settings` → `Build Options` → `Enable Bitcode`

### Multiple commands produce '...libyoga.a'

| Xcode: `Product` (top menu) → `Archive`

```
Multiple commands produce '...libyoga.a':
1) Target 'yoga' has a command with output '...libyoga.a'
2) Target 'yoga' has a command with output '...libyoga.a'

Multiple commands produce '...libReact.a':
1) Target 'React' has a command with output '...libReact.a'
2) Target 'React' has a command with output '...libReact.a'
```

**solution**

1. <https://github.com/facebook/react-native/issues/20492#issuecomment-422958184>

> <https://github.com/facebook/react-native/issues/20492#issuecomment-422958184>
>
> The following is needed to ensure the "archive" step works in XCode. It
> removes React & Yoga from the Pods project, as it is already included in
> the main project.
>
> Without this, you'd see errors when you archive like:
> "Multiple commands produce ... libReact.a"
> "Multiple commands produce ... libyoga.a"

```ruby
# ios/Podfile

post_install do |installer|
  installer.pods_project.targets.each do |target|
    targets_to_ignore = %w[React yoga]

    if targets_to_ignore.include?(target.name)
      target.remove_from_project
    end
  end
end
```

```sh
$ cd ios
$ pod install
```

Android errors
--------------

NOTE: all emulator options are omitted for brevity in examples below.

### Could not determine java version from '9.0.4'

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

***UPDATE (2019-07-11)***

`java8` formula is removed from `homebrew/cask-versions` repository - install
`adoptopenjdk8` formula instead:

```sh
$ brew tap homebrew/cask-versions
$ brew cask install adoptopenjdk8
```

### Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

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

### No such file or directory - /usr/local/share/android-sdk

1. <https://github.com/caskroom/homebrew-cask/issues/32139>

```sh
$ brew cask uninstall android-sdk
Error: No such file or directory - /usr/local/share/android-sdk
```

**solution**

```sh
$ ln -s /usr/local/Caskroom/android-sdk/25.2.3 /usr/local/share/android-sdk
$ brew cask uninstall android-sdk
```

### repositories.cfg could not be loaded

1. <https://askubuntu.com/questions/885658>

```sh
$ sdkmanager --list
Warning: File /Users/tap/.android/repositories.cfg could not be loaded.
```

**solution**

```sh
$ touch ~/.android/repositories.cfg
```

### truncated package paths in output from sdkmanager

1. <https://stackoverflow.com/questions/42460205>

```sh
$ sdkmanager --list
...
Available Packages:
  Path                              | Version      | Description
  -------                           | -------      | -------
  add-ons;addon-g..._apis-google-15 | 3            | Google APIs
  add-ons;addon-g..._apis-google-16 | 4            | Google APIs
```

**solution**

```sh
$ sdkmanager --list --verbose
```

***UPDATE (2019-06-07)***

`sdkmanager` doesn't truncate package paths anymore.

### Qt library not found

1. <https://stackoverflow.com/questions/40931254>

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
[140735129182208]:ERROR:./android/qt/qt_setup.cpp:28:Qt library not found at ../emulator/lib64/qt/lib
Could not launch '../emulator/qemu/darwin-x86_64/qemu-system-x86_64': No such file or directory
```

**solution**

it's necessary to cd to emulator directory and use relative path:

```sh
$ cd /usr/local/share/android-sdk/emulator
$ ./emulator -avd Nexus_5X_API_23_x86_64
```

or else create an alias in _~/.zshenv_:

```conf
export ANDROID_HOME=/usr/local/share/android-sdk

alias emulator='cd $ANDROID_HOME/emulator && ./emulator'
alias avd='emulator -avd Nexus_5X_API_23_x86_64'
```

***UPDATE***

after updating emulator to version 26.1.4.0 it's no longer necessary to cd to
emulator directory but still you cannot rely on `PATH` - use absolute path:

_~/.zshenv_:

```conf
alias avd='$ANDROID_HOME/tools/emulator -avd Nexus_5X_API_23_x86_64'
# `Qt library not found` again:
#alias avd='emulator -avd Nexus_5X_API_23_x86_64'
```

### Cannot find AVD system path

NOTE: the error occurs sporadically and might disappear on its own.

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
PANIC: Cannot find AVD system path. Please define ANDROID_SDK_ROOT
```

**solution**

_~/.zshenv_:

```conf
export ANDROID_SDK_ROOT=/usr/local/share/android-sdk
```

### The SDK directory does not exist

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> The SDK directory '/Users/tap/Library/Android/sdk' does not exist.
```

**solution**

if you previously used Android Studio to run application it might have created
_android/local.properties_ file in project with the following content:

```conf
sdk.dir=/Users/tap/Library/Android/sdk
```

that is RN now expects Android SDK to be installed in _~/Library/Android/sdk/_
(this is where Android Studio would install it).

there are 2 ways to solve the problem:

- create symlink that points to actual Android SDK root:

  ```sh
  $ mkdir ~/Library/Android
  $ ln -s $ANDROID_HOME ~/Library/Android/sdk
  ````

  or edit _android/local.properties_ directly:

  ```
  # export ANDROID_HOME=/usr/local/share/android-sdk
  sdk.dir=/usr/local/share/android-sdk
  ```

- just comment out or remove that line

  RN will search for Android SDK in `$ANDROID_HOME` then.

### Could not get unknown property 'MYAPP_RELEASE_STORE_FILE'

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* Where:
Build file '<APP_DIR>/android/app/build.gradle' line: 108

* What went wrong:
A problem occurred evaluating project ':app'.
> Could not get unknown property 'MYAPP_RELEASE_STORE_FILE' for SigningConfig_Decorated...
```

**solution**

1. <https://facebook.github.io/react-native/docs/signed-apk-android.html>

create dummy _~/.gradle/gradle.properties_ file:

```conf
MYAPP_RELEASE_STORE_FILE=test
MYAPP_RELEASE_KEY_ALIAS=test
MYAPP_RELEASE_STORE_PASSWORD=test
MYAPP_RELEASE_KEY_PASSWORD=test
```

since these variables (`MYAPP_RELEASE_STORE_FILE`, etc.) are mentioned in
_android/app/build.gradle_ project file and build fails if they were not set.

### DeviceException: No connected devices!

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:installDebug'.
> com.android.builder.testing.api.DeviceException: No connected devices!
```

**solution**

start emulator before running application.

### Could not find com.facebook.react:react-native

1. <https://github.com/oblador/react-native-vector-icons/issues/480>
2. <https://github.com/facebook/react-native/issues/14223#issuecomment-304447493>

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > A problem occurred configuring project ':react-native-push-notification'.
      > Could not resolve all dependencies for configuration ':react-native-push-notification:_debugPublishCopy'.
         > Could not find com.facebook.react:react-native:0.42.0.
           Required by:
               myapp:react-native-push-notification:unspecified
```

**solution**

add this snippet to _android/build.gradle_ and specify actual RN version in it
(make sure to update this value every time RN version changes):

```gradle
allprojects {
  configurations.all {
    resolutionStrategy {
      eachDependency { DependencyResolveDetails details ->
        if (details.requested.group == 'com.facebook.react' && details.requested.name == 'react-native') {
            details.useVersion '0.42.3' # specify actual RN version here
        }
      }
    }
  }
}
```

### CANNOT TRANSLATE guest DNS ip

1. <https://github.com/facebook/react-native/issues/13340>

```sh
$ emulator -avd Nexus_5X_API_23_x86_64
...
CANNOT TRANSLATE guest DNS ip
```

**solution**

issue is resolved after updating emulator to version 26.1.4.0:

```sh
$ sdkmanager --update
```

### no application launcher icon on home screen in emulator

1. <https://developer.android.com/guide/practices/ui_guidelines/icon_design_launcher.html>

application doesn't have launcher icon on home screen in emulator.

**solution**

`react-native run-android` installs application but doesn't add application
launcher icon on home screen - you can do it manually:

- go to applications
- click and hold application icon
- drag application icon to home screen

### Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE

```
$ react-native run-android
...
com.android.ddmlib.InstallException: Failed to finalize session : INSTALL_FAILED_VERSION_DOWNGRADE
```

**solution**

1. <https://stackoverflow.com/questions/13808599>

RN fails to install application with lower versionCode than currently installed
in emulator - uninstall application with higher versionCode inside emulator and
build again.

### You have not accepted the license agreements of the following SDK components

```
$ react-native run-android
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > A problem occurred configuring project ':react-native-svg'.
      > You have not accepted the license agreements of the following SDK components:
        [Android SDK Build-Tools 25.0.1, Android SDK Platform 25].
        Before building your project, you need to accept the license agreements and complete the installation of the missing components using the Android Studio SDK Manager.
        Alternatively, to learn how to transfer the license agreements from one workstation to another, go to http://d.android.com/r/studio-ui/export-licenses.html
```

**solution**

```sh
$ sudo sdkmanager --licenses
/ accept all licenses
```

### Configuration with name 'default' not found.

```
$ react-native run-android
...
Building and installing the app on the device (cd android && ./gradlew installDebug)...

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring project ':app'.
> Could not resolve all dependencies for configuration ':app:_debugApk'.
   > Configuration with name 'default' not found.
```

**solution**

I haven't installed new dependencies after rebase:

```
$ npm install
```

### Manifest merger failed

```
$ react-native run-android
...
> Task :app:processDebugManifest FAILED
<APP_DIR>/android/app/src/debug/AndroidManifest.xml:22:18-91 Error:
        Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91
        is also present at [androidx.core:core:1.0.0] AndroidManifest.xml:22:18-86 value=(androidx.core.app.CoreComponentFactory).
        Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:7:6-11:8 to override.

See http://g.co/androidstudio/manifest-merger for more information about the manifest merger.


FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:processDebugManifest'.
> Manifest merger failed : Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91
        is also present at [androidx.core:core:1.0.0] AndroidManifest.xml:22:18-86 value=(androidx.core.app.CoreComponentFactory).
        Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:7:6-11:8 to override.
```

**solution**

> <https://medium.com/@yathousen/the-day-google-decided-to-shake-the-react-native-community-4ba5cdd33388>
>
> It was June 17th, 2019, Android released their new package names for their
> DEPRECATED support libraries + other tools inside AndroidX...
>
> Basically, all of the basic libraries that Google provide for the Android +
> React Native Community, on the last or close to last versions were affected.
> In some cases, it was the fault of the react-native libraries maintainers
> for using imports like + symbols that are ambiguous and generally take the
> last version available of the library and in some other cases was caused by
> the fact that the developer was actually using new features from the latest
> release.

> <https://stackoverflow.com/a/52517772/3632318>
>
> AndroidX is redesigned library to make package names more clear.

> <https://stackoverflow.com/a/55849025/3632318>
>
> AndroidX is a major improvement to the original Android Support Library.
>
> AndroidX fully replaces the Support Library by providing feature parity and
> new libraries.

> <https://stackoverflow.com/a/54533702/3632318>
>
> One app should use either AndroidX or old Android Support libraries. That's
> why you faced this issue.
>
> So the solution is to use either AndroidX or old Support Library.

so the problem is that application must be using both new AndroidX and old
Android Support libraries - hence the error.

it's possible to examine all application dependencies using this command:

```sh
$ cd android
$ ./gradlew app:dependencies
```

AFAIU there can be either `com.android.support` or `androidx.*` libraries in
the output but not both.

the reason why AndroidX libraries leak into current application dependencies
is that many RN libraries use the latest versions of Google libraries instead
of specific ones, say (note `+` symbol):

```groovy
// https://github.com/zo0r/react-native-push-notification/blob/c4c961cf3a76bffe4e9e36cf00201611eecbb898/android/build.gradle

dependencies {
    // ...
    implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
    // ...
    implementation "com.google.firebase:firebase-messaging:${safeExtGet('firebaseVersion', '+')}"
}
```

but the latest versions of Google started to depend on AndroidX libraries since
June 17th, 2019 and it was a breaking change.

there might be several solutions to this problem:

#### [RECOMMENDED] try upgrading RN packages

1. <https://github.com/react-native-community/async-storage/issues/128#issuecomment-505423631>

maybe maintainers of RN packages have fixed their _build.gradle_ files to use
specific versions of Google libraries:

```sh
$ yarn upgrade --pattern react-native
```

it was the case with `react-native-device-info` package:

> <https://github.com/react-native-community/react-native-device-info/pull/693>
>
> With the latest firebase/gcm release, the latest version (+) now brings in
> androidx dependencies. react-native won't be ready for androidx until 0.60
> so constrain the gcm version to 16.1.0 (the latest pre-androidx version).

```diff
  # https://github.com/react-native-community/react-native-device-info/pull/693/files#diff-7ae5a9093507568eabbf35c3b0665732

  dependencies {
    implementation "com.facebook.react:react-native:${safeExtGet('reactNativeVersion', '+')}"
-   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
+   implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '16.1.0')}"
```

#### fork RN packages

1. <https://github.com/react-native-community/react-native-device-info/pull/693>

if RN package is not actively maintained it might take a long time till it's
fixed, so it makes sense to fork and fix it by yourself (just like it's done
in the linked PR which is also mentioned previously).

NOTE: you can open a PR to the upstream repository from a fork.

#### force use of specific versions of Google libraries

1. <https://github.com/facebook/react-native/issues/25292#issuecomment-502998885>
2. <https://medium.com/@suchydan/how-to-solve-google-play-services-version-collision-in-gradle-dependencies-ef086ae5c75f>
3. <https://github.com/facebook/react-native/issues/25307#issuecomment-503516458>
4. <https://medium.com/mindorks/avoiding-conflicts-in-android-gradle-dependencies-28e4200ca235>

in this solution it's not necessary to modify Google libraries but your own
_android/app/build.gradle_ file only.

perform these steps for each RN package which depends on AndroidX libraries
until there are no AndroidX dependencies (run `gradlew app:dependencies` to
find such RN packages as described above):

- find Google libraries in RN package dependencies

  libraries inside `com.google.android.gms` and `com.google.firebase` groups
  are affected:

  ```groovy
  // https://github.com/zo0r/react-native-push-notification/blob/c4c961cf3a76bffe4e9e36cf00201611eecbb898/android/build.gradle

  dependencies {
      // ...
      implementation "com.google.android.gms:play-services-gcm:${safeExtGet('googlePlayServicesVersion', '+')}"
      // ...
      implementation "com.google.firebase:firebase-messaging:${safeExtGet('firebaseVersion', '+')}"
  }
  ```

- exclude these libraries from RN package dependencies in your application

  ```diff
    // android/app/build.gradle

    dependencies {
  -     implementation project(':react-native-push-notification')
  +     implementation(project(':react-native-push-notification')) {
  +       exclude group: 'com.google.android.gms', module: 'play-services-gcm'
  +       exclude group: 'com.google.firebase', module: 'firebase-messaging'
  +     }
  ```

- use specific versions of these libraries in your application

  ```diff
    // android/app/build.gradle

    dependencies {
        // ...
  +     implementation("com.google.android.gms:play-services-base:16.1.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-basement:16.2.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-gcm:16.1.0") {
  +       force = true;
  +     }
  +     implementation("com.google.android.gms:play-services-stats:16.0.1") {
  +       force = true;
  +     }
  +     implementation("com.google.firebase:firebase-messaging:18.0.0") {
  +       force = true;
  +     }
  +     implementation("com.google.firebase:firebase-iid:18.0.0") {
  +       force = true;
  +     }
    }
  ```

  a rule of thumb is not to use June versions, the last but one versions will
  do in most cases.

  see [Maven repository](https://mvnrepository.com/) for the list of library
  versions and corresponding release dates.

  it might be necessary to force versions of both direct and transitive RN
  package dependencies (just like in example above) - inspect the output of
  `gradlew app:dependencies` command after adding all direct dependencies.

### Could not get unknown property 'android' for root project 'iceperkapp' of type org.gradle.api.Project

1. <https://documentation.onesignal.com/docs/react-native-sdk-setup#section-adding-the-gradle-plugin>

```
$ cd android
$ ./gradlew assembleRelease
...
FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'iceperkapp'.
> Could not get unknown property 'android' for root project 'iceperkapp' of type org.gradle.api.Project.
```

the error occurred right after configuring `react-native-onesignal` package by
following official `React Native SDK Setup` guide (see the link).

**solution**

official guide contains error:

> <https://github.com/geektimecoil/react-native-onesignal/issues/550#issuecomment-396015932>
>
> you applied the onesignal-gradle-plugin to your root build.gradle or
> android/build.gradle instead of the one in app/build.gradle

```diff
  // android/build.gradle

- apply plugin: 'com.onesignal.androidsdk.onesignal-gradle-plugin'
```

```diff
  // android/app/build.gradle

+ apply plugin: 'com.onesignal.androidsdk.onesignal-gradle-plugin'
```

### ENOTEMPTY: directory not empty

```
$ cd android
$ ./gradlew assembleRelease
...
:app:bundleReleaseJsAndAssets
Scanning 773 folders for symlinks in <APP_DIR>/node_modules (41ms)
Scanning 773 folders for symlinks in <APP_DIR>/node_modules (37ms)
Loading dependency graph, done.

ENOTEMPTY: directory not empty, rmdir '/var/folders/1g/28pm7dxn0_l569vjyzlzp2zw0000gn/T/react-native-packager-cache-f18bd0fb39fa7507ecdd2fb6cd91757d41b78c44/cache'

:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssets'.
> Process 'command 'node'' finished with non-zero exit value 1
```

**solution**

reset cache:

```sh
$ react-native start --reset-cache
```

### resource android:style/TextAppearance.Material.Widget.Button.Colored not found

```
$ cd android
$ ./gradlew assembleRelease
...
> Task :react-native-exit-app:verifyReleaseResources FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':react-native-exit-app:verifyReleaseResources'.
> 1 exception was raised by workers:
  com.android.builder.internal.aapt.v2.Aapt2Exception: Android resource linking failed
  error: resource android:style/TextAppearance.Material.Widget.Button.Borderless.Colored not found.
  error: resource android:style/TextAppearance.Material.Widget.Button.Colored not found.
  <APP_DIR>/node_modules/react-native-exit-app/android/build/intermediates/res/merged/release/values-v26/values-v26.xml:7: error: resource android:attr/colorError not found.
```

the error is usually accompanied by this warning when configuring corresponding
project:

```
$ cd android
$ ./gradlew assembleRelease
...
> Configure project :react-native-exit-app
WARNING: Configuration 'compile' is obsolete and has been replaced with 'implementation' and 'api'.
It will be removed at the end of 2018. For more information see: http://d.android.com/r/tools/update-dependency-configurations.html
WARNING: The specified Android SDK Build Tools version (25.0.0) is ignored, as it is below the minimum supported version (28.0.3) for Android Gradle Plugin 3.4.0.
Android SDK Build Tools 28.0.3 will be used.
To suppress this warning, remove "buildToolsVersion '25.0.0'" from your build.gradle file, as each version of the Android Gradle Plugin now has a default version of the build tools.
```

**solution**

explanation:

> <https://stackoverflow.com/a/49332191/3632318>
>
> android:style/TextAppearance.Material.Widget.Button.Borderless.Colored was
> added in API 24 so you can't use it with version 23. You can use a style that
> was added before version 23.

in brief RN package is using component which is not available in API level 23
(this API level is specified in _android/build.gradle_ file of that RN package).

=> increase API level supported by RN package (either fork RN packaged yourself
or find a fork where it has been already done):

```diff
  // android/build.gradle

  android {
-   compileSdkVersion 23
-   buildToolsVersion "23.0.1"
+   compileSdkVersion 28
+   buildToolsVersion "28.0.1"

    defaultConfig {
        minSdkVersion 16
-       targetSdkVersion 22
+       targetSdkVersion 28
    }
```

> <https://stackoverflow.com/a/24523113/3632318>
>
> compileSdkVersion is the API version of Android that you compile against.
>
> buildToolsVersion is the version of the compilers (aapt, dx, renderscript
> compiler, etc...) that you want to use. For each API level (starting with 18),
> there is a matching .0.0 version.

> <https://developer.android.com/studio/releases/gradle-plugin.html#3-2-0>
>
> 3.2.1 (October 2018)
>
> With this update, you no longer need to specify a version for the SDK Build
> Tools. The Android Gradle plugin now uses version 28.0.3 by default.

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

`react-native-git-upgrade` command downgraded `react` package to another alpha
version and this is what most likely fixed the issue.

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

### method does not override or implement a method from a supertype

```
$ react-native run-android
...
<APP_DIR>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerPackage.java:19: error: method does not override or implement a method from a supertype
  @Override
  ^
Note: <APP_DIR>/node_modules/react-native-image-picker/android/src/main/java/com/imagepicker/ImagePickerActivityEventListener.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
1 error
:react-native-image-picker:compileReleaseJavaWithJavac FAILED

FAILURE: Build failed with an exception.
```

**solution**

for each package that fails to compile:

- bump its version in _package.json_ (preferably to the latest one)
- `npm install`
- `npm updated <package_name>`

if it doesn't help, try to unlink/link failing package manually:

```sh
$ react-native unlink <package_name>
$ react-native link <package_name>
```

errors after upgrading RN to 0.52.1
-----------------------------------

### Error: While resolving module `react-native-vector-icons/Octicons`

```
$ react-native start
...
error: bundling failed: Error: While resolving module `react-native-vector-icons/Octicons`, the Haste package `react-native-vector-icons` was found. However the module `Octicons` could not be found within the package. Indeed, none of these files exist:

  * `<APP_DIR>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
  * `<APP_DIR>/node_modules/react-native/local-cli/core/__fixtures__/files/Octicons/index(.native||.ios.js|.native.js|.js|.ios.json|.native.json|.json)`
```

**solution**

1. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-357405396>
2. <https://github.com/oblador/react-native-vector-icons/issues/626#issuecomment-358018994>

```sh
$ rm ./node_modules/react-native/local-cli/core/__fixtures__/files/package.json
$ react-native start
```

it looks like this file is recreated after each application build so it makes
sense to add this command to `postinstall` script in _package.json_:

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

```
$ react-native run-ios
...
<APP_DIR>/node_modules/react-native-flurry-analytics/ios/RNFlurryAnalytics.m:2:9: fatal error: 'Flurry.h' file not found
#import <Flurry.h>
        ^~~~~~~~~~
1 error generated.
```

**solution**

see [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}) for the tip
on how to repair CocoaPods.

### object file (.../libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)

```
$ react-native run-ios
...
ld: warning: object file (<APP_DIR>/ios/build/Build/Products/Debug-iphonesimulator/libAutoGrowTextInput.a(AutogrowTextInputManager.o)) was built for newer iOS version (9.3) than being linked (8.0)
```

**solution**

1. <https://stackoverflow.com/a/32950454/3632318>

`react-native-autogrow-textinput` is built with `IPHONEOS_DEPLOYMENT_TARGET=9.3`
build setting but the rest of the project (the library is linked to later) has
been built with `IPHONEOS_DEPLOYMENT_TARGET=8.0` build setting => hence the error.

the latest version of `react-native-autogrow-textinput` doesn't support iOS
Deployment Target (DT) lower than 9.3 but current DT in iOS project is 8.0 =>
change DT in iOS project to 9.3 as well:

| Xcode: `Build Settings` → `Deployment` → `iOS Deployment Target`

### Undefined symbols for architecture x86_64

```
$ react-native run-ios
...
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_RCTOneSignal", referenced from:
      objc-class-ref in AppDelegate.o
  "_OBJC_CLASS_$_RNSentry", referenced from:
      objc-class-ref in AppDelegate.o
ld: symbol(s) not found for architecture x86_64
```

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/18#issuecomment-287132994>
2. <https://facebook.github.io/react-native/docs/linking-libraries-ios.html#manual-linking>

link `react-native-onesignal` and `react-native-sentry` libraries manually and
rebuild application.

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

the error occurs when evaluating this line:

```javascript
// node_modules/react-native-device-info/deviceinfo.js

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

```
$ cd android
$ ./gradlew assembleRelease
...
sentry-cli update to 1.28.1 is available!
run sentry-cli update to update
error: http error: [55] Failed sending data to the peer (SSL_write() returned SYSCALL, errno = 32)
:app:bundleReleaseJsAndAssets FAILED

FAILURE: Build failed with an exception.

* Where:
Script '<APP_DIR>/node_modules/react-native-sentry/sentry.gradle' line: 154

* What went wrong:
Execution failed for task ':app:bundleReleaseJsAndAssetsreleaseSentryUpload'.
> Process 'command 'node_modules/sentry-cli-binary/bin/sentry-cli'' finished with non-zero exit value 1
```

**solution**

```sh
$ npm update react-native-sentry
$ npm_reset
```

upgrading `react-native-sentry` didn't help - cleaning cache and reinstalling
all node modules fixed the issue.

### production release crashes on startup (in both Android and iOS devices)

**solution**

1. <https://github.com/facebook/react-native/issues/16567#issuecomment-340041357>
2. <https://github.com/facebook/react-native/issues/16542>

<https://github.com/magus/react-native-facebook-login/issues/279>:

> View.propTypes has been removed in 0.49

replace `View.propTypes.style` with `ViewPropTypes.style` in all components.

see also [React Native - Android]({% post_url 2017-05-24-react-native-android %})
(`debugging` section) on how to debug similar problems on Android.

errors after upgrading RN from 0.52.1 to 0.59.9
-----------------------------------------------

### react-native-push-notification: Appears to be a git repo or submodule.

```
$ npm install
...
npm ERR! path <APP_DIR>/node_modules/react-native-push-notification
npm ERR! code EISGIT
npm ERR! git <APP_DIR>/node_modules/react-native-push-notification: Appears to be a git repo or submodule.
npm ERR! git     <APP_DIR>/node_modules/react-native-push-notification
npm ERR! git Refusing to remove it. Update manually,
npm ERR! git or move it out of the way first.
```

**solution**

1. <https://github.com/zo0r/react-native-push-notification/issues/1057>

```sh
$ rm -rf node_modules/*/.git
```

### incompatible types: ReadableArray cannot be converted to ReadableNativeArray

```
$ react-native run-android
...
> Task :react-native-sentry:compileDebugJavaWithJavac FAILED
<APP_DIR>/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java:251: error: incompatible types: Read
ableArray cannot be converted to ReadableNativeArray
            addExceptionInterface(eventBuilder, exception.getString("type"), exception.getString("value"), stacktrace.getArray("frames"));
                                                                                                                              ^
Note: <APP_DIR>/node_modules/react-native-sentry/android/src/main/java/io/sentry/RNSentryModule.java uses or overrides a deprecated
 API.
```

**solution**

```sh
$ npm install --save react-native-sentry@0.43.1
```

NOTE: 0.43.1 is the latest version at the time of writing.

### constructor RNSentryPackage in class RNSentryPackage cannot be applied to given types

```
$ react-native run-android
...
> Task :app:compileDebugJavaWithJavac FAILED
<APP_DIR>/android/app/src/main/java/com/iceperkapp/MainApplication.java:45: error: constructor RNSentryPackage in class RNSentryPac
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

```
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
Unfortunately, iceperkapp has stopped
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
Unfortunately, iceperkapp has stopped
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

### Plugin/Preset files are not allowed to export objects, only functions.

```
$ react-native start
...
Loading dependency graph, done.
error: bundling failed: Error: Plugin/Preset files are not allowed to export objects, only functions. In <APP_DIR>/node_modules/babel-preset-flow/lib/index.js
    at createDescriptor (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:178:11)
    at <APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:109:50
    at Array.map (<anonymous>)
    at createDescriptors (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:109:29)
    at createPresetDescriptors (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:101:10)
    at presets (<APP_DIR>/node_modules/@babel/core/lib/config/config-descriptors.js:47:19)
    at mergeChainOpts (<APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:320:26)
    at <APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:283:7
    at mergeExtendsChain (<APP_DIR>/node_modules/@babel/core/lib/config/config-chain.js:299:21)
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

```
$ react-native start
...
error: bundling failed: Error: The 'decorators' plugin requires a 'decoratorsBeforeExport' option, whose value must be a boolean. If you are migrating from Babylon/Babel 6 or want to use the old decorators proposal, you should use the 'decorators-legacy' plugin instead of 'decorators'.
    at validatePlugins (<APP_DIR>/node_modules/@babel/parser/lib/index.js:6093:13)
    at getParser (<APP_DIR>/node_modules/@babel/parser/lib/index.js:11286:5)
    at parse (<APP_DIR>/node_modules/@babel/parser/lib/index.js:11256:22)
    at parser (<APP_DIR>/node_modules/@babel/core/lib/transformation/normalize-file.js:170:34)
    at normalizeFile (<APP_DIR>/node_modules/@babel/core/lib/transformation/normalize-file.js:138:11)
    at runSync (<APP_DIR>/node_modules/@babel/core/lib/transformation/index.js:44:43)
    at transformSync (<APP_DIR>/node_modules/@babel/core/lib/transform.js:43:38)
    at Object.transform (<APP_DIR>/node_modules/metro-react-native-babel-transformer/src/index.js:202:20)
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

### We ran "xcodebuild" command but it exited with error code 65

```
$ react-native run-ios
...
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening iceperkapp.xcworkspace
```

**solution**

this can be anything so:

> To debug build logs further, consider building your app with Xcode.app, by
> opening iceperkapp.xcworkspace

### Duplicate module name: react-native

```
$ react-native start
...
Loading dependency graph...(node:10228) UnhandledPromiseRejectionWarning: Error: jest-haste-map: Haste module naming collision:
  Duplicate module name: react-native
  Paths: <APP_DIR>/ios/Pods/React/package.json collides with <APP_DIR>/node_modules/react-native/package.json

This error is caused by `hasteImpl` returning the same name for different files.
    at setModule (<APP_DIR>/node_modules/jest-haste-map/build/index.js:569:17)
    at workerReply (<APP_DIR>/node_modules/jest-haste-map/build/index.js:641:9)
    at processTicksAndRejections (internal/process/task_queues.js:89:5)
    at async Promise.all (index 83)
(node:10228) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 2)
(node:10228) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
(node:10228) UnhandledPromiseRejectionWarning: Error: jest-haste-map: Haste module naming collision:
  Duplicate module name: react-native
  Paths: <APP_DIR>/ios/Pods/React/package.json collides with <APP_DIR>/node_modules/react-native/package.json
```

**solution**

1. <https://facebook.github.io/react-native/docs/integration-with-existing-apps#configuring-cocoapods-dependencies>
2. <https://github.com/react-native-community/react-native-svg/issues/621#issuecomment-473346430>

incorrect version of `React` pod was installed into _ios/Pods/_ as a dependency
of `RNCAsyncStorage` pod:

```ruby
# node_modules/@react-native-community/async-storage/RNCAsyncStorage.podspec

Pod::Spec.new do |s|
  s.name         = "RNCAsyncStorage"
  # ...

  s.dependency 'React'
end
```

there are 2 problems here:

- version of `React` pod is incorrect (0.11.0 but should be 0.59.9)
- `React` pod shouldn't be installed into _ios/Pods/_ at all (see below why)

now fix both problems:

- configure CocoaPods dependencies correctly

  see [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}) for the
  tip on how to configure CocoaPods dependencies.

- remove `React` pod from _ios/Pods/_ directory

  since `RNCAsyncStorage` pod has `path` option in _ios/Podfile_ (which points to
  corresponding package location in _node\_modules/_ directory), it shouldn't be
  installed into _ios/Pods/_ - as well as its dependencies (that is `React` pod).

  but directory for `React` pod is still there and _ios/Pods/React/package.json_
  collides with _node\_modules/react-native/package.json_ as both have the same
  `name` field - `react-native`.

  it happened because `pod install` doesn't remove unused pods from _ios/Pods/_
  => it should be done manually by deintegrating your project:

  ```
  $ cd ios
  $ pod deintegrate
  Deintegrating `iceperkapp.xcodeproj`
  Deleted 1 'Copy Pods Resources' build phases.
  Deleted 1 'Check Pods Manifest.lock' build phases.
  - libPods-iceperkapp.a
  - Pods-iceperkapp.debug.xcconfig
  - Pods-iceperkapp.release.xcconfig
  Deleting Pod file references from project
  - libPods-iceperkapp-tvOSTests.a
  - libPods-iceperkappTests.a
  Deleted 1 empty `Pods` groups from project.
  Removing `Pods` directory.

  Project has been deintegrated. No traces of CocoaPods left in project.
  Note: The workspace referencing the Pods project still remains.
  ```

  or just by removing _ios/Pods/_ directory:

  ```sh
  $ rm -rf ios/Pods
  ```

  finally install pods again:

  ```sh
  $ cd ios
  $ pod install
  ```

### no known class method for selector 'didReceiveRemoteNotification:'

build inside Xcode failed:

```
<APP_DIR>/ios/iceperkapp/AppDelegate.m:71:17: error: no known class method for selector 'didReceiveRemoteNotification:'
  [RCTOneSignal didReceiveRemoteNotification:notification];
                ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

**solution**

1. <https://github.com/geektimecoil/react-native-onesignal/issues/421#issuecomment-373877766>

```diff
  // ios/iceperkapp/AppDelegate.m

- // Required for the notification event.
- - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)notification {
-   [RCTOneSignal didReceiveRemoteNotification:notification];
- }
-
```

### node_modules/sentry-cli-binary/bin/sentry-cli: No such file or directory

build inside Xcode failed:

```
/Users/tap/Library/Developer/Xcode/DerivedData/iceperkapp-fqktxncrxxusoqcuegfuetzddexv/Build/
  Intermediates.noindex/iceperkapp.build/Debug-iphonesimulator/iceperkapp.build/
  Script-00DD1BFF1BD5951E006B06BC.sh: line 4: ../node_modules/sentry-cli-binary/bin/sentry-cli:
  No such file or directory
Command PhaseScriptExecution failed with a nonzero exit code
```

**solution**

find and replace all occurrences of `sentry-cli-binary` with `@sentry/cli` in
your project.

### The iOS Simulator deployment target is set to 7.0

```
$ react-native run-ios
...
info warning: The iOS Simulator deployment target is set to 7.0, but the range of supported deployment target versions for this platform is 8.0 to 12.2.99. (in target 'InAppUtils')
```

**solution**

1. <https://github.com/CocoaPods/CocoaPods/issues/8069>

> <https://github.com/react-native-community/react-native-maps/issues/2638#issue-393483116>
>
> It appears the newest version of Xcode doesn't support a deployment target
> less than 8.0.

> <https://github.com/CocoaPods/CocoaPods/issues/7314#issuecomment-422368382>
>
> if the target is built for 9.0, then the Pods should build for 9.0, not for
> 8.0.
>
> It's nice to want to support more stuff, but it should come as fixes to the
> actual CocoaPods gem, not as complex workarounds in people's Podfile.

> <https://github.com/CocoaPods/CocoaPods/issues/7314#issuecomment-494659918>
>
> platform
>
> Specifies the platform for which a static library should be built, but NOT the
> deployment target for which it should be built (the number in platform :ios,
> '9.0' is only to find a compatible version of the pod). The actual deployment
> target will be lower than specified, will have unused symbols and will cause
> you a lot of headache.

```ruby
# node_modules/react-native-in-app-utils/react-native-in-app-utils.podspec

Pod::Spec.new do |s|
  # ...
  s.platform        = :ios, "7.0"
end
```

```ruby
# ios/Podfile

platform :ios, '9.0'
```

Podspecs and Podfile both specify deployment target for which a static library
should be built. still deployment target in Podfile is not the minimum allowed
deployment target => deployment target from Podspec might be used which will
cause warnings like above if that deployment target is not supported by Xcode.

=> ignore this warning for now.

### withRef is removed

emulator window:

```
Unhandled JS Exception: withRef is removed. To access the wrapped instance,
use a ref on the connected component
```

**solution**

1. <https://react-redux.js.org/api/connect#forwardref-boolean>

> <https://github.com/reduxjs/react-redux/releases/tag/v6.0.0>
>
> The withRef option to connect has been replaced with forwardRef. If
> {forwardRef : true} has been passed to connect, adding a ref to the connected
> wrapper component will actually return the instance of the wrapped component.

> <https://github.com/reduxjs/react-redux/issues/1291#issuecomment-494188070>
>
> Basically, change withRef to forwardRef, and then delete the .getWrappedInstance()
> part of componentRef.getWrappedInstance(). componentRef is your own component now.

say:

```diff
  // MyComponent.js

  export default connect(
    mapStateToProps,
    null,
    null,
-   {withRef: true},
+   {forwardRef: true},
  )(MyComponent);
```

usage:

```diff
  <MyComponent ref={ref => this._myComponentRef = ref} />

  // ...
- this._myComponentRef.getWrappedInstance().save();
+ this._myComponentRef.save();
```
