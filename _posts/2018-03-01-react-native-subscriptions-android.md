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

implement subscription in application
-------------------------------------

1. <https://github.com/idehub/react-native-billing>

test subscription
-----------------

1. <https://developer.android.com/google/play/billing/billing_testing.html#testing-subscriptions>

subscriptions in test environment:

- have shortened periods (say, 1 month → 5 minutes)
- renew 6 times only (cancelled afterwards)

### testing with static responses

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

#### set license key to `null`

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

#### about error codes

1. <https://developer.android.com/google/play/billing/billing_reference.html>
2. <https://github.com/idehub/react-native-billing/issues/17#issuecomment-228036758>
3. <https://github.com/anjlab/android-inapp-billing-v3/blob/master/library/src/main/java/com/anjlab/android/iab/v3/Constants.java>
4. <https://github.com/idehub/react-native-billing/blob/master/android/src/main/java/com/idehub/Billing/InAppBillingBridge.java>

- standard response codes from Google are 0-8
- `react-native-billing` is wrapping `android-inapp-billing-v3` library
  which may return its own error codes (> 100)

in both cases `react-native-billing` rejects promise with the same error
message containing returned error code:

```
Purchase or subscribe failed with error: <error_code>
```

### testing on real device

see [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %}).

it's required to run application on real device even for testing:

> Error: InAppBilling is not available. InAppBilling will not work/test on
> an emulator, only a physical Android device.

prepare release with subscription
---------------------------------

### include license key in application build

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
