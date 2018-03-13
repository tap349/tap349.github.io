---
layout: post
title: React Native - Subscriptions
date: 2018-03-01 18:40:21 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

iOS
---

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>
2. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>

### create in-app purchase (IAP)

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `In-App Purchases ⨁`

> Select the in-app purchase you want to create.

- [x] `Auto-Renewable Subscription`

> Create Auto-Renewable Subscription

- `Reference Name`: `No Ads Monthly`
- `Product ID`: `<bundle_id>.sub.noads.monthly`

  `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
  in _project.pbxproj_ - `com.iceperk.iceperkapp` for our application.

  product ID must be unique - even after deleting subscription
  its product ID cannot be reused for new subscriptions.

> Create Subscription Group

- [x] `Create New Subscription Group`
  - `Subscription Group Reference Name`: `No Ads`

    must be more general than subscription reference name.

### configure IAP

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `<subscription>` (link)

- [x] `Cleared for Sale`
- `Subscription Duration` → `1 Month`
- `Subscription Prices ⨁`
  - `Currency`: `RUB - Russian Ruble`
  - `Price`: `149 р.`
- `Localizations ⨁` → `Russian`
  - `Subscription Display Name`: `Скрытие рекламы на месяц`
  - `Description`: don't add so far (maybe not required)

<https://developer.apple.com/library/content/technotes/tn2259/_index.html#//apple_ref/doc/uid/DTS40009578-CH1-ITUNES_CONNECT>:

> Leave the state of your product as Missing Metadata.
>
> Upload a a screenshot of your in-app purchase product once you
> are done testing it and ready to upload it for review.

### configure subscription group

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `<subscription_group>` (link)

it's required to add localization for subscription group as well:

> Before you can submit your in-app purchase for review,
> you must add at least one localization to your subscription group.

- `Localizations ⨁` → `Russian`
  - `Subscription Group Display Name`: `Подписки`
  - `App Name Display Name`: [x] Use App Name

### create sandbox tester (test user)

1. <https://support.magplus.com/hc/en-us/articles/203809008-iOS-How-to-Test-In-App-Purchases-in-Your-App>

| iTunes Connect: `My Apps` → `Users and Roles` → `Sandbox Testers` (tab)
| `Testers ⨁` (link)

NOTE: test account == sandbox tester account.

tips and notes regarding test account:

- new sandbox tester shouldn't have an existing Apple ID

  I got `That email address is already associated with an existing Apple ID`
  error when I tried to add myself as a sandbox tester - Apple ID is created
  for each new sandox tester right after it's added here.

- sign out of Apple ID on iPhone before testing subscription

  it's also important not to sign in as a sandbox tester in `Settings` but
  in application only (when prompted before making a purchase) - a sandbox
  tester account is not a fully functional one and tester's Apple ID cannot
  be used to access iTunes Store or App Store.

  occasionally `Apple ID Verification` alert dialog might appear asking to
  enter password for test account in `Settings` - it's safe to ignore it and
  press `Not Now` button all the time.

#### about verification of sandbox tester email

generally speaking it's necessary to verify new test account by following
the link sent to specified email:

> You must validate your e-mail, or any purchases you make will silently fail.

but I couldn't verify account of a new sandbox tester - after clicking
the link email verification page kept on loading forever with an activity
indicator in the middle of the screen and JS errors in Chrome Console:

```
bundle.js:46270 Refused to create a worker from 'blob:https://id.apple.com/...'
because it violates the following Content Security Policy directive...
```

IDK if it matters or not but I signed in to iCloud with this account
(in browser) and completed registration by adding 3 secret questions.

in the end I could make a purchase using not verified test account.

### implement subscription in application

1. <https://github.com/chirag04/react-native-in-app-utils>
2. <https://github.com/sibelius/iap-receipt-validator>
3. [Adding In-App Purchase to Your Applications](https://developer.apple.com/library/content/technotes/tn2259/_index.html)
4. [Validating Receipts With the App Store](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html)
5. [Receipt Fields](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html)

#### obtain a shared secret

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases (left menu)` → `App-Specific Shared Secret` → `View Master Shared Secret` (link)

shared secret is used to validate receipts with `iap-receipt-validator`
npm package.

### test subscription

subscriptions in test environment:

- have shortened periods (say, 1 month → 5 minutes)
- renew 5 times only (cancelled afterwards)
- cannot be managed (say, cancelled manually)

<https://forums.developer.apple.com/thread/14702>:

> the sandbox environment handles auto-renewing subscriptions in an automated
> fashion - that is subscription periods are shortened and renew only 5 times
> and not controlled through the subscription management screen.

#### run application on real device

see [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %}).

it's required to run application on real device to make purchases:
the only action allowed from inside emulator is loading products.

### prepare release with subscription

#### add new IAP when submitting new app version

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases (left menu)`

> Your first in-app purchase must be submitted with a new app version.
> Select it from the app’s In-App Purchases section and click Submit.
>
> Once your binary has been uploaded and your first in-app purchase has
> been submitted for review, additional in-app purchases can be submitted
> using the table below.

Android
-------

### implement subscription in application

1. <https://github.com/idehub/react-native-billing>

### test subscription

1. <https://developer.android.com/google/play/billing/billing_testing.html#testing-subscriptions>

subscriptions in test environment:

- have shortened periods (say, 1 month → 5 minutes)
- renew 6 times only (cancelled afterwards)

#### set license key to `null`

1. <https://github.com/idehub/react-native-billing#testing-with-static-responses>

use `null` license key when testing with static responses
(using reserved product IDs - `android.test.purchased`, etc.).

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

I got this error when tried to test subscriptions with static responses.

in the end I did factory data reset to get rid of this error - though it might
be sufficient just to remove/add Google account (see also other advice in the
article mentioned).

#### Error: This version of the application is not configured for billing through Google Play

I got this error when tried to subscribe using reserved product ID
(`android.test.purchased`).

it has turned that you can only purchase when testing with static responses but
not subscribe. all other subscription related methods from `react-native-billing`
package (`isSubscribed`, `getSubscriptionDetails`) are working as expected (that
is they return relevant static responses - for subscriptions, not for purchases).

#### run application on real device

see [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %}).

it's required to run application on real device even for testing:

> Error: InAppBilling is not available. InAppBilling will not work/test on
> an emulator, only a physical Android device.

### prepare release with subscription

#### include license key in application build

1. <https://support.google.com/googleplay/android-developer/answer/186113?hl=en>

> Licensing allows you to prevent unauthorized distribution of your app.
> It can also be used to verify in-app billing purchases.

| Google Play Console: `All applications` → `<my_app>`
| `Development tools` (left menu) → `Services & APIs` → `Licensing & in-app billing`

_android/app/src/main/res/values/strings.xml_:

```diff
  <resources>
      <string name="app_name">Хоккей</string>
+     <string name="RNB_GOOGLE_PLAY_LICENSE_KEY">YOUR_GOOGLE_PLAY_LICENSE_KEY_HERE</string>
  </resources>
```
