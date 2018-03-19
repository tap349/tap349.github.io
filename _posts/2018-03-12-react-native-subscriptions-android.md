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

- IAB - In-app Billing
- GP - Google Play
- GPC - Google Play Console

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

GPC:

> Licensing allows you to prevent unauthorized distribution of your app.
> It can also be used to verify in-app billing purchases.

1. <https://support.google.com/googleplay/android-developer/answer/186113?hl=en>
2. <https://developer.android.com/google/play/billing/billing_admin.html#license_key>

| GPC: `All applications` → `<my_app>`
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

add subscription in GPC
-----------------------

### create signed APK

1. [React Native - Releases]({% post_url 2017-12-26-react-native-releases %})
2. <https://facebook.github.io/react-native/docs/signed-apk-android.html>

assemble production release with IAB permission as usual.

NOTE: you cannot upload APK with the same build number twice.

### upload APK to alpha

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
(see `test with a test subscription` section).

| GPC: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `Alpha` (section) → `MANAGE ALPHA` (button)

- `CREATE RELEASE` (button)
- `BROWSE_FILES` (button) → add new APK
- `Release name` (input): `3.14.alpha` (for example)
- `What's new in this release?` (textarea): `Добавили подписку` (it's for alpha only)
- `SAVE` (button) → `REVIEW` (button)

### add pricing template

1. <https://support.google.com/googleplay/android-developer/answer/138000?hl=en&ref_topic=3452890>

| GPC: `Settings` (left menu)
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
      (see `upload APK to alpha` section).

| GPC: `All applications` → `<my_app>`
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
  until it's published (see `test with a real subscription` section).

- `Pricing` (combobox): `Import from pricing template` →
  `RUB 149.00 - Подписка на месяц` → `IMPORT` (button)
- `Billing period` (combobox): `Monthly`
- `Free trial period` (input): empty (will be set to `0`)
- `Grace period` (radiobutton): `7 DAYS` (default)
- `SAVE` (button)

test subscription on real device
--------------------------------

1. <https://developer.android.com/google/play/billing/billing_testing.html#testing-subscriptions>
2. <https://stackoverflow.com/a/22469253/3632318> (list of requirements)
3. <http://suda.pl/the-hell-of-testing-google-play-in-app-billing/>
4. <https://www.youtube.com/watch?v=XMrUBC4Xgl8>

emulator (app):

> InAppBilling is not available. InAppBilling will not work/test on
> an emulator, only a physical Android device.

### testing with static responses

YOU DON'T NEED TO TOUCH GOOGLE PLAY CONSOLE AT ALL TO TEST WITH STATIC RESPONSES.

1. <https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-static>
2. <https://github.com/idehub/react-native-billing#testing-with-static-responses>

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

#### run application on real device

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

#### troubleshooting

- `Authentication is required. You need to sign in to your Google Account`

  - where: standard Google popup in application
  - when: testing subscription with static responses

  1. <https://www.androidpit.com/how-to-fix-google-play-authentication-is-required-error>

  in the end I did factory data reset to get rid of the error - though it might
  be sufficient just to remove/add Google account (see also other advice in the
  article mentioned).

- `This version of the application is not configured for billing through Google Play`

  - where: standard Google popup in application
  - when: trying to subscribe using reserved product ID (`android.test.purchased`)

  it has turned out you can only purchase when testing with static responses but
  not subscribe. all other subscription related methods from `react-native-billing`
  package (`isSubscribed`, `getSubscriptionDetails`) are working as expected (that
  is they return relevant static responses - for subscription, not for purchase).

  <https://github.com/idehub/react-native-billing#testing-with-your-own-in-app-products>:

  > Testing with static responses is limited, because you are only able to test
  > the purchase function.

  <http://suda.pl/the-hell-of-testing-google-play-in-app-billing/>:

  > Subscriptions can't be tested using static responses...

### testing with test subscriptions

1. <https://developer.android.com/google/play/billing/billing_testing.html#test-purchases-sandbox>

test subscriptions are:

- purchased with a test card (test trasactions are created)
- have shortened periods (say, 1 month → 5 minutes)
- renew 6 times only (cancelled automatically afterwards)
- can be managed (say, cancelled in GP)

test subscriptions are not created separately - they are real subscriptions
but with special characteristics and behaviour for license testers.

=> it's required to create and publish a real subscription in iTunes
Connect as described above (application itself may stay unpublished).

<https://developer.android.com/google/play/billing/billing_admin.html#billing-form-add>:

> To be visible to a user during checkout, an item's publishing state must be
> set to Active, and the item's app must be published on Google Play.
>
> If you're using a test account, users can see active items within unpublished
> apps, as well.

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> A test account can purchase an item in your product list only if the item is
> published.

#### add license testers (license test accounts, license tester accounts)

1. <https://developer.android.com/google/play/billing/billing_testing.html#setup>
2. <https://developer.android.com/google/play/billing/billing_admin.html#billing-testing-setup>

| GPC: `Settings` (left menu)
| `Developer account` (left menu) → `Account details` → `License Testing` (section)

- `Gmail accounts with testing access` (textarea): `*.tap349@gmail.com`

it's not required that license test account is linked to a valid payment
method (I could make a test purchase without any linked payment method).

if user is registered as both license and alpha tester, being a license
tester has priority: he'll be offered to purchase test subscription only.

NOTE: license test account must be a primary account on real device!

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> The only way to change the primary account on a device is to do a factory
> reset, making sure you log on with your primary account first.

#### set valid license key

see `include license key in application build` section.

#### uninstall existing application from real device

it's necessary to uninstall application from device beforehand
or else you'll get either `INSTALL_FAILED_DUPLICATE_PERMISSION`
or `INSTALL_FAILED_ALREADY_EXISTS` error:

```sh
$ adb uninstall com.iceperkapp
```

NOTE: `com.iceperkapp` is both package name and `applicationId`
      from _android/app/build.gradle_.

#### set either development or production environment in application

it's possible to purchase test subscriptions in both `development`
and `production` environments (see the next section) - just don't
forget to forward ports in `development`.

#### run or install application on real device

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

NOTE: application (being installed now) and APK (uploaded to GPC)
      must have the same version name and build number.

TODO: check previous statement and find its source.

- run application in debug mode

  - `__DEV__` variable is set to true
  - you can debug JS remotely

  ```sh
  $ react-native run-android
  ```

- run application in release mode

  - `__DEV__` variable is set to false
  - you cannot debug JS remotely

  ```sh
  $ react-native run-android --variant release
  ```

- install APK on real device

  1. <https://developer.android.com/studio/build/building-cmdline.html#RunningOnDevice>

  same as running application in release mode:

  - `__DEV__` variable is set to false
  - you cannot debug JS remotely

  ```sh
  $ adb -d install android/app/build/outputs/apk/app-release.apk
  ```

#### view financial reports

test transactions are not shown in financial reports. IDK how to view them -
currently I can track them through GP emails only (they are sent to license
test account email each time subscription is renewed or cancelled).

#### troubleshooting

- `The item you requested is not available for purchase.`

  - where: standard Google popup in application
  - when: trying to purchase a subscription

  current primary account is not a license test account.

- `The publisher cannot purchase the item.`

  - where: standard Google popup in application
  - when: trying to purchase a subscription

  you cannot use developer account to test real subscriptions -
  even with test transactions (and of course you cannot make real
  purchases since you cannot pay to yourself).

- `isSubscribed` returns true after subscription is cancelled

  1. <https://github.com/idehub/react-native-billing/issues/62>
  2. <https://github.com/idehub/react-native-billing/issues/57>

  when test subscription is cancelled, `isSubscribed` keeps on returning
  true. reinstalling application and refreshing subscription status cache
  with `loadOwnedPurchasesFromGoogle` doesn't help - most likely there is
  nothing wrong with my implementation and `react-native-billing` package:

  > if im not wrong, thats the way PlayStore works, its cycle can take
  > days to remove the "Subscribed" item from user PlayStore/Account.
  > Even coding in Android, after canceled the subscription take some
  > days to disappear.

  possible solution (but not an easy one) is to use server-side validation
  of subscriptions with `in-app-purchase` package.

  even though `isSubscribed` returns true, subscription is not actually
  active - you can purchase it again (though it's button to purchase is
  hidden inside application because of `isSuscribed` returning true).

  UPDATE: `isSubscribed` is still true 12 hours later.
  UPDATE: `isSubscribed` is eventually false 2 days later.

### testing with real subscriptions

1. <https://developer.android.com/google/play/billing/billing_testing.html#transactions>
2. <https://github.com/idehub/react-native-billing#testing-with-your-own-in-app-products>

<https://developer.android.com/google/play/billing/billing_testing.html#billing-testing-test>:

> You can do end-to-end testing of your app by publishing it to an alpha
> distribution channel. This allows you to publish the app to the Google
> Play Store, but limit its availability to just the testers you designate.

#### add alpha testers (alpha test accounts, alpha tester accounts)

release review summary when trying to publish release to alpha without testers:

> This release will not be available to any users because you haven't specified
> any testers for it yet.

<https://developer.android.com/google/play/billing/billing_testing.html#transactions>:

> With alpha/beta test groups, real users (chosen by you) can install your app
> from Google Play and test your in-app products. They can make real purchases
> that result in actual charges to their accounts, using any of their normal
> payment methods in Google Play to make purchases. Note that if you include
> test license accounts in your alpha and beta distribution groups, those users
> will only be able to make test purchases.

=> license testers cannot make real purchases!

so I have removed myself from license testers (`*.tap349@gmail.com` email) and
added to alpha testers instead (see below). also unlike for license test account
it's important that alpha test account is linked to a valid payment method - so
make sure to add payment info for this account before trying to make a purchase.

| GPC: `Settings` (left menu)
| `Manage testers` (left menu) → `CREATE LIST` (button)

- `List name` (input): `team`
- `Add email addresses` (input): `*.tap349@gmail.com`
- `CREATE LIST` (button)

| GPC: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `MANAGE ALPHA` (button) → `Manage testers` (section)

- `Choose a testing method` (combobox): `Closed Alpha Testing`
- `Users`: [x] `team` (activate the list)
- `SAVE` (button)

it's possible to add/remove alpha testers after release is published.

#### publish release to alpha

NOTE: publish release to alpha only if you're planning to start Alpha Testing
      (either open or closed) - don't publish if you will test application by
      installing and running it on real device with `adb`.

| GPC: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `MANAGE ALPHA` (button)

- `EDIT RELEASE` (button)
- upload new release if necessary
- `SAVE` (button) → `REVIEW` (button) → `START ROLLOUT TO ALPHA` (button)

#### open opt-in URL in browser

| GPC: `All applications` → `<my_app>`
| `Release management` (left menu) → `App releases` → `MANAGE ALPHA` (button) → `Manage testers` (section) → `Opt-in URL` (input)

- open link in browser
- `BECOME A TESTER` (button)
- `Download the <my_app> app on Google Play` (link)
- install application from Google Play Store
- try to make a purchase

#### view financial reports

| GPC: `All applications` → `<my_app>`
| `Financial reports` (left menu) → `Subscriptions`

statistics are collected with 2 days delay (there're no data for the last 2 days).

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

### update tax info (if necessary)

| GPC: `Settings` (left menu)
| `Developer account` (left menu) → `Payment settings` → `Settings` (section) → `MANAGE SETTINGS` (button)
| `Payments profile` (section) → `United States tax info` → `✎` (link) → `UPDATE TAX INFO` (link)

> Tax form questions

- `Are you a US citizen, US resident alien, US corporation or US partnership?` (radiobutton): [x] `No`
- `CONTINUE` (button)

> Certificate of Foreign Status

- `Classification` (combobox): `LLC`
- `Signature of beneficial owner (or individual authorized to sign for beneficial owner):` (input): `T* Sm*`
- `SUBMIT` (button)
