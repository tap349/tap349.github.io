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

IAB - In-app Billing

implement subscription in application
-------------------------------------

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

Google Play Console:

> Licensing allows you to prevent unauthorized distribution of your app.
> It can also be used to verify in-app billing purchases.

<https://support.google.com/googleplay/android-developer/answer/186113?hl=en>:

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

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %}).
2. <https://developer.android.com/google/play/billing/billing_testing.html#testing-subscriptions>
3. <http://suda.pl/the-hell-of-testing-google-play-in-app-billing/>
4. <https://www.youtube.com/watch?v=XMrUBC4Xgl8>

emulator (app):

> Error: InAppBilling is not available. InAppBilling will not work/test on
> an emulator, only a physical Android device.

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

### testing with real subscriptions

TODO

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
