---
layout: post
title: React Native - Subscriptions (Android)
date: 2018-03-01 18:40:21 +0300
access: public
comments: true
categories: [react-native, android]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://developer.android.com/google/play/billing/index.html>
2. <https://developer.android.com/google/play/billing/billing_subscriptions.html>

IAB - In-app Billing

add subscription in Google Play Console
---------------------------------------

### create signed APK

1. [React Native - Releases]({% post_url 2017-12-26-react-native-releases %})
2. <https://facebook.github.io/react-native/docs/signed-apk-android.html>

assemble production release as usual:

- change environment to `production`
- change version number and increment build number

  NOTE: you cannot upload APK with the same build number twice.

- assemble release

  ```sh
  $ build_android_release='cd android && ./gradlew assembleRelease; cd ..'
  $ build_android_release
  ```

### upload APK to alpha channel

<https://developer.android.com/google/play/billing/billing_admin.html#billing-list-setup>:

> The link to the In-app Products page appears only if you have a
> Google payments merchant account and the app's manifest includes
> the com.android.vending.BILLING permission.

NOTE: in fact `In-app products` page is available even when application with
      IAB permission is not uploaded but there are no `MANAGED PRODUCTS` and
      `SUBSCRIPTIONS` tabs to manage corresponding items in product list.

=> upload APK with IAB permission to alpha channel to be able to set up
subscription (actually upload APK anywhere - alpha channel is just the
safest option).

at the same time it's not required to publish (rollout) application - you
can both set up subscription and test it on real device without publishing
(see `test with real subscriptions` section).

| Google Play Console: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `Alpha` (section) → `EDIT RELEASE` (button)

- `BROWSE_FILES` (button) → add new APK
- `Release name` (input): `3.14.alpha` (for example)
- `What's new in this release?` (textarea): `Добавили подписку` (it's for alpha only)
- `SAVE` (button) → `REVIEW` (button)

### add pricing template

1. <https://support.google.com/googleplay/android-developer/answer/138000?hl=en&ref_topic=3452890>

| Google Play Console: `Settings` (left menu)
| `Pricing templates` (left menu) → `NEW PRICING TEMPLATE` (button)

- `Name` (input): `Подписка на месяц`
- `Price` (input): `149`
- `Tax`: [x] `Add applicable tax on top of price`
- `SAVE` (button)

about `Add applicable tax on top of price` tax option:

> Local tax is added in addition to set price in select countries. Local
> prices are generated using exchange rates and local pricing patterns.
> Tax is only added in countries with set tax rates.

=> EUR 2.09 (~ RUB 149) becomes EUR 2.49 (~ RUB 179).

### set up subscription (add subscription item to product list)

1. <https://developer.android.com/google/play/billing/billing_admin.html#billing-form-add>

NOTE: you can't set up subscription until you upload APK with IAB permission
      (see `upload APK to alpha channel` section).

| Google Play Console: `All applications` → `<my_app>`
| `Store presence` (left menu) → `In-app products` → `SUBSCRIPTIONS` (tab) → `CREATE SUBSCRIPTION` (button)

- `Product ID` (input): `com.iceperk.iceperkapp.sub.noads.monthly`
- `Title` (input): `Скрытие рекламы на месяц`
- `Description` (input): `Полное отключение рекламы в приложении`
- `Status` (radiobutton): `ACTIVE`

  setting `ACTIVE` status means subscription is published (activated) after
  it's created - once subscription is published, it can't be deactivated or
  deleted, neither can you modify the price or billing period afterwards.

  you can opt to publish subscription later but still it must be done
  eventually - no account (even test one) can purchase a subscription
  until it's published (see `test with real subscriptions` section).

- `Pricing` (combobox): `Import from pricing template` →
  `RUB 149.00 - Подписка на месяц` → `IMPORT` (button)
- `Billing period` (combobox): `Monthly`
- `Free trial period` (input): empty (will be set to `0`)
- `Grace period` (radiobutton): `7 DAYS` (default)
- `SAVE` (button)

implement subscriptions in application
--------------------------------------

1. <https://github.com/idehub/react-native-billing>

<https://developer.android.com/google/play/billing/api.html#subs>:

> Unlike managed products, subscriptions cannot be consumed.

### include license key in application build

<https://developer.android.com/google/play/billing/billing_integrate.html#billing-security>:

> To help ensure the integrity of the transaction information that is sent to
> your application, Google Play signs the JSON string that contains the response
> data for a purchase order. Google Play uses the private key that is associated
> with your application in the Play Console to create this signature. The Play
> Console generates an RSA key pair for each application.
>
> To find the public key portion of this key pair, open your application's
> details in the Play Console, click 'Services & APIs', and review the field
> titled 'Your License Key for This Application'.
>
> When your application receives this signed response, you can use the public
> key portion of your RSA key pair to verify the signature.

Google Play Console:

> Licensing allows you to prevent unauthorized distribution of your app.
> It can also be used to verify in-app billing purchases.

1. <https://support.google.com/googleplay/android-developer/answer/186113?hl=en>
2. <https://developer.android.com/google/play/billing/billing_admin.html#license_key>

| Google Play Console: `All applications` → `<my_app>`
| `Development tools` (left menu) → `Services & APIs` → `Licensing & in-app billing`

_android/app/src/main/res/values/strings.xml_:

```diff
  <resources>
      <string name="app_name">Хоккей</string>
+     <string name="RNB_GOOGLE_PLAY_LICENSE_KEY">YOUR_GOOGLE_PLAY_LICENSE_KEY_HERE</string>
  </resources>
```

### declare IAB permission

<https://developer.android.com/google/play/billing/billing_integrate.html#billing-permission>:

> In-app billing relies on the Google Play application, which handles all of the
> communication between your application and the Google Play server. To use the
> Google Play application, your application must request the proper permission.
>
> If your application does not declare the In-app Billing permission, but attempts
> to send billing requests, Google Play refuses the requests and responds with an error.

_android/app/src/main/AndroidManifest.xml_:

```diff
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
+ <uses-permission android:name="com.android.vending.BILLING" />
  <uses-feature android:name="android.hardware.camera" android:required="false"/>
```

test subscription on real device
--------------------------------

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})
2. <https://developer.android.com/google/play/billing/billing_testing.html#testing-subscriptions>
3. <http://suda.pl/the-hell-of-testing-google-play-in-app-billing/>
4. <https://www.youtube.com/watch?v=XMrUBC4Xgl8>

emulator (app):

> Error: InAppBilling is not available. InAppBilling will not work/test on
> an emulator, only a physical Android device.

subscriptions in test environment:

- have shortened periods (say, 1 month → 5 minutes)
- renew 6 times only (cancelled afterwards)

### test with static responses

YOU DON'T NEED TO TOUCH GOOGLE PLAY CONSOLE AT ALL TO TEST WITH STATIC RESPONSES.

1. <https://github.com/idehub/react-native-billing#testing-with-static-responses>
2. <https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-static>

> You do not need to list the reserved products in your application's product list.
> Google Play already knows about the reserved product IDs. Also, you do not need
> to upload your application to the Play Console to perform static response tests
> with the reserved product IDs. You can simply install your application on a device,
> log into the device, and make billing requests using the reserved product IDs.

reserved product IDs:

- android.test.purchased
- android.test.canceled
- android.test.refunded
- android.test.item_unavailable

#### set license key to `null` temporarily

1. <https://github.com/idehub/react-native-billing#testing-with-static-responses>

use `null` license key when testing with static responses.

_android/app/src/main/res/values/strings.xml_:

```diff
  <resources>
      <string name="app_name">Хоккей</string>
+     <string name="RNB_GOOGLE_PLAY_LICENSE_KEY" />
  </resources>
```

however you cannot remove `RNB_GOOGLE_PLAY_LICENSE_KEY` property
altogether - or else you'll get `String resource ID #0x0` error.

#### Error: Authentication is required. You need to sign in to your Google Account

1. <https://www.androidpit.com/how-to-fix-google-play-authentication-is-required-error>

I got this error when I tried to test subscription with static responses.

in the end I did factory data reset to get rid of the error - though it might
be sufficient just to remove/add Google account (see also other advice in the
article mentioned).

#### Error: This version of the application is not configured for billing through Google Play

I got this error when I tried to subscribe using reserved product ID
(`android.test.purchased`).

it has turned that you can only purchase when testing with static responses but
not subscribe. all other subscription related methods from `react-native-billing`
package (`isSubscribed`, `getSubscriptionDetails`) are working as expected (that
is they return relevant static responses - for subscription, not for purchase).

<https://github.com/idehub/react-native-billing#testing-with-your-own-in-app-products>:

> Testing with static responses is limited, because you are only able to test
> the purchase function.

<http://suda.pl/the-hell-of-testing-google-play-in-app-billing/>:

> Subscriptions can't be tested using static responses...

### test with real subscriptions

1. <https://github.com/idehub/react-native-billing#testing-with-your-own-in-app-products>

it's required to create and publish a real subscription in iTunes Connect as
described above before you proceed (application itself can stay unpublished).

<https://developer.android.com/google/play/billing/billing_admin.html#billing-form-add>:

> To be visible to a user during checkout, an item's publishing state must be
> set to Active, and the item's app must be published on Google Play.
>
> If you're using a test account, users can see active items within unpublished
> apps, as well.

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> A test account can purchase an item in your product list only if the item is
> published.

#### add license test accounts

1. <https://developer.android.com/google/play/billing/billing_testing.html#setup>
2. <https://developer.android.com/google/play/billing/billing_admin.html#billing-testing-setup>

| Google Play Console: `Settings` (left menu)
| `Account details` (left menu) → `License Testing` (section)

- `Gmail accounts with testing access` (textarea): `*.tap349@gmail.com`

NOTE: license test account must be a primary account on real device!

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> The only way to change the primary account on a device is to do a factory
> reset, making sure you log on with your primary account first.

#### run application on real device

1. <https://developer.android.com/studio/build/building-cmdline.html#RunningOnDevice>

```sh
$ adb uninstall com.iceperkapp
$ adb -d install android/app/build/outputs/apk/app-release.apk
```

NOTE: package name is `applicationId` from _android/app/build.gradle_.

it's necessary to uninstall application from device beforehand
or else you'll get either `INSTALL_FAILED_DUPLICATE_PERMISSION`
or `INSTALL_FAILED_ALREADY_EXISTS` error.

#### [OPTIONAL] publish release to alpha channel

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> You can do end-to-end testing of your app by publishing it to an alpha
> distribution channel. This allows you to publish the app to the Google
> Play Store, but limit its availability to just the testers you designate.

NOTE: publish release to alpha only if you're planning to start Alpha Testing
      (either open or closed) - don't publish if you will test application by
      installing and running it on real device with `adb`.

release review summary when trying to publish release to alpha without testers:

> This release will not be available to any users because you haven't specified
> any testers for it yet.

TODO: add testers in `Manage testers`

| Google Play Console: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `MANAGE ALPHA` (button)

TODO: make new testers active and publish release to alpha

### subscription error codes

1. <https://developer.android.com/google/play/billing/billing_reference.html>
2. <https://github.com/idehub/react-native-billing/issues/17#issuecomment-228036758>
3. <https://github.com/anjlab/android-inapp-billing-v3/blob/master/library/src/main/java/com/anjlab/android/iab/v3/Constants.java>
4. <https://github.com/idehub/react-native-billing/blob/master/android/src/main/java/com/idehub/Billing/InAppBillingBridge.java>

- standard response codes from Google are 0-8
- `react-native-billing` is wrapping `android-inapp-billing-v3` library which
  may return its own error codes (> 100)

in both cases `react-native-billing` rejects promise with this error message:

```
Purchase or subscribe failed with error: <error_code>
```

prepare release with subscription
---------------------------------

TODO
