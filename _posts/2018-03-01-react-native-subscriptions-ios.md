---
layout: post
title: React Native - Subscriptions (iOS)
date: 2018-03-01 18:40:21 +0300
access: public
comments: true
categories: [react-native, ios]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. [Workflow for configuring in-app purchases](https://help.apple.com/itunes-connect/developer/#/devb57be10e7)
2. [In-App Purchase Programming Guide](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Introduction.html)
3. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>
4. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>

implement subscriptions in application
--------------------------------------

1. <https://github.com/chirag04/react-native-in-app-utils>
2. <https://github.com/sibelius/iap-receipt-validator>
3. [Adding In-App Purchase to Your Applications](https://developer.apple.com/library/content/technotes/tn2259/_index.html)
4. [Validating Receipts With the App Store](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html)
5. [Receipt Fields](https://developer.apple.com/library/content/releasenotes/General/ValidateAppStoreReceipt/Chapters/ReceiptFields.html)

### obtain a shared secret

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases (left menu)` → `App-Specific Shared Secret` → `View Master Shared Secret` (link)

shared secret is used to validate receipts with `iap-receipt-validator`
npm package.

add subscription in iTunes Connect
----------------------------------

### create in-app purchase (IAP)

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `In-App Purchases` (section) → `⨁` (link)

> Select the in-app purchase you want to create.

- [x] `Auto-Renewable Subscription`

> Create Auto-Renewable Subscription

- `Reference Name` (input): `No Ads Monthly`
- `Product ID` (input): `<bundle_id>.sub.noads.monthly`

  `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
  in _project.pbxproj_ - `com.iceperk.iceperkapp` for our application.

  product ID must be unique - even after deleting subscription
  its product ID cannot be reused for new subscriptions.

> Create Subscription Group

- [x] `Create New Subscription Group`
  - `Subscription Group Reference Name` (input): `No Ads`

    must be more general than subscription reference name.

### configure IAP

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `<subscription>` (link)

- [x] `Cleared for Sale`
- `Subscription Duration` (combobox): `1 Month`
- `Subscription Prices` (section) → `⨁` (link)
  - `Currency` (combobox): `RUB - Russian Ruble`
  - `Price` (combobox): `149 р.`
- `Localizations` (section) → `⨁` (link) → `Russian`
  - `Subscription Display Name` (input): `Скрытие рекламы на месяц`
  - `Description` (input): `Полное отключение рекламы в приложении` (not required)

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

- `Localizations` (section) → `⨁` (link) → `Russian`
  - `Subscription Group Display Name` (input): `Подписки`
  - `App Name Display Name`: [x] Use App Name

test subscription on real device
--------------------------------

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

it's required to run application on real device to make purchases:
the only action allowed from inside emulator is loading products.

test subscriptions:

- have shortened periods (say, 1 month → 5 minutes)
- renew 5 times only (cancelled automatically afterwards)
- cannot be managed (say, cancelled manually)

test subscriptions are not created separately - they are real subscriptions
but with special characteristics and behaviour sandbox testers.

<https://forums.developer.apple.com/thread/14702>:

> the sandbox environment handles auto-renewing subscriptions in an automated
> fashion - that is subscription periods are shortened and renew only 5 times
> and not controlled through the subscription management screen.

### create sandbox tester (test user)

1. <https://support.magplus.com/hc/en-us/articles/203809008-iOS-How-to-Test-In-App-Purchases-in-Your-App>

| iTunes Connect: `My Apps` → `Users and Roles` → `Sandbox Testers` (tab)
| `Testers` (section) → `⨁` (link)

NOTE: test account == sandbox tester account.

tips and notes regarding test account:

- new sandbox tester shouldn't have an existing Apple ID

  I got `That email address is already associated with an existing Apple ID`
  error when I tried to add myself as a sandbox tester - Apple ID is created
  for each new sandox tester right after it's added here.

- sign out of Apple ID on iPhone before testing subscription

  1. <https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/ShowUI.html#//apple_ref/doc/uid/TP40008267-CH3-SW11>

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

**UPDATE (2018-03-19)**

I have succesfully verified email of another sandbox tester -
most likely it was temporary technical problem on Apple side.

### set either development or production environment in application

it's possible to purchase test subscriptions in both `development` and
`production` environments but read the caveats about validating receipt
from sandbox tester in `production` environment in the next section.

NOTE: there are lots of `Network request failed` errors when trying
      to purchase test subscription - so keep on trying :)

### run application on real device

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

sandbox tester can purchase a subscription in production release but
subscription status cannot be checked because sandbox receipt cannot
be validated in `production` environment (see `troubleshooting` section
below for details).

### troubleshooting

- `App installation failed. An unknown error has occurred.`

  where: | Xcode modal window
  when:  | running application on real device

  <https://stackoverflow.com/a/41138265/3632318>:

  - close Xcode
  - unplug device
  - uninstall application from device
  - open Xcode
  - clean build in Xcode
  - run application on device again

- `not_available`

  - where: `Log.error`
  - when: checking subscription status

  there is no signed in Apple ID => no receipt is found.

  wait for `Sign-In Required` modal window or sign in manually in
  iPhone's `Settings`.

- `Sandbox receipt sent to Production environment`

  - where: `Log.error`
  - when: checking subscription status

  current environment is `production` and it's passed to `iapReceiptValidator`
  function of `iap-receipt-validator` package when trying to validate receipt
  of sandbox tester.

  so switch to `development` environment or send `development` environment
  to `iapReceiptValidator` function regardless of current environment.

- checking subscription status always fails in production release

  this is exactly the previous error - you cannot validate receipt of
  sandbox tester in `production` environment.

  and this is why Apple reviewer couldn't test subscription:

  - he is signed in as sandbox tester on his iPhone
  - he uses production release - so it's a `production` environment
  - by default subscription status is `true` (before any checks)
  - checking subscription status always fails for sandbox tester in
    production release
  - subscription status remains to be `true`
  - reviewer doesn't see the button to subscribe and decides that
    subscription functionality is not implemented
  - binary is rejected in iTunes Connect :(

  solution? I guess production releases shouldn't be tested by sandbox testers at
  all since we can't tell sandbox tester from real user inside our application -
  it's not our application user but currently signed in Apple ID on iPhone itself.
  so we always have to assume `production` environment when validating receipt in
  production release.

  the only thing we can do here is to change behaviour of our application and
  set default subscription status to `false` so that button to subscribe is
  shown by default - all subsequent subscription status checks will fail so
  the button will be always shown until sandbox tester makes a purchase.
  then subscription status should change to `true` though since this is done
  inside operation to purchase a subscription - not within operation to check
  subscription status which keeps on failing forever. and it's all okay until
  the user decides to restart application - then subscription status is reset
  to its default value `false` which won't change even though subscription has
  been just purchased.

  I guess all of this should be properly taken down in review notes for IAP in
  iTunes Connect.

  <https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/AppReview.html>:

  > When validating receipts on your server, your server needs to be able
  > to handle a production-signed app getting its receipts from Apple’s test
  > environment. The recommended approach is for your production server
  > to always validate receipts against the production App Store first. If
  > validation fails with the error code “Sandbox receipt used in production”,
  > validate against the test environment instead.

prepare release with subscription
---------------------------------

### fill review information

1. <https://stackoverflow.com/questions/4802810/iphone-in-app-purchase-screen-shot/4803012>
2. <https://developer.apple.com/app-store/review/guidelines/>
3. <https://help.apple.com/itunes-connect/developer/#/dev84b80958f>

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases` (left menu) → `<subscription>` (link) → `Review Information` (section)

- `Screenshot`: screenshot of your app with subscription popup
- `Review Notes`: details on how you work with subscription in your app

  notes might also include details about how you're going to monetize your
  application and some implementation details (the more the better I guess).

- `Save` (button)

after you fill review information, subscription status is automatically
changed from `Missing Metadata` to `Ready to Submit`.

### create new app version

`In-App Purchases` page:

> Your first in-app purchase must be submitted with a new app version.
> Select it from the app’s In-App Purchases section and click Submit.
>
> Once your binary has been uploaded and your first in-app purchase has
> been submitted for review, additional in-app purchases can be submitted
> using the table below.

| iTunes Connect: `My Apps` → `<my_app>` → `App Store` (tab)
| `⨁ VERSION OR PLATFORM` (link in left menu) → `iOS` (popup menu)

- `Store Version Number`: new version number (say, `3.14`)
- `Create` button

new app version has `Prepare for Submission` status now.

### add IAP to new app version

1. <https://stackoverflow.com/a/42191936/3632318>

| iTunes Connect: `My Apps` → `<my_app>` → `App Store` (tab)
| `3.14 Prepare for Submission` (left menu) → `In-App Purchases` (section) → `⨁` (link)

NOTE: `In-App Purchases` section is available only if you've created IAP before.

- select IAP (your subscription) in popup window

  > Select in-app purchases for us to review with this app version.
  > The in-app purchases shown below are the only ones in the Ready to Submit state.

- `Done` (button to select IAP and close popup window)
- `Save` (button to save changes to app version)

#### troubleshooting

- binary rejected (release didn't pass a review because of IAP)

| iTunes Connect: `My Apps` → `<my_app>` → `Activity` (tab)
| `App Store Versions` (left menu) → `Resolution Center` (link)

> **2.** 1 Performance: App Completeness
> <br>
> **3.** 1.2 Business: Payments - Subscriptions
>
> <h3>Guideline 2.1 - Performance - App Completeness</h3>
>
> We found that while you have submitted in-app purchase products for your app,
> the in-app purchase functionality is not present in your binary.
>
> **Next Steps**
>
> If you would like to include in-app purchases in your app, you will need to
> upload a new binary that incorporates the in-app purchase API to enable users
> to make a purchase.
>
> Once you revise and resubmit your binary, you will also need to resubmit your
> in-app purchases for review since they are in the Developer Action Required
> state. For each in-app purchase product submitted, please be sure to edit the
> detail information or cancel the request to change the detail information for
> the in-app purchases using iTunes Connect.
>
> <h3>Guideline 3.1.2 - Business - Payments - Subscriptions</h3>
>
> We noticed that your app or its metadata did not fully meet the terms and
> conditions for auto-renewing subscriptions, as specified in Schedule 2,
> section 3.8(b) of the Paid Applications agreement.
>
> Your app's **binary** did not include:
>
> <ul>
> <li> The following information about the auto-renewable nature of the subscription
>   <ul>
>   <li> Title of publication or service </li>
>   <li> Length of subscription (time period and content or services provided
>        during each subscription period) </li>
>   <li> Price of subscription, and price per unit if appropriate </li>
>   <li> Payment will be charged to iTunes Account at confirmation of purchase </li>
>   <li> Subscription automatically renews unless auto-renew is turned off at
>        least 24-hours before the end of the current period </li>
>   <li> Account will be charged for renewal within 24-hours prior to the end of
>        the current period, and identify the cost of the renewal </li>
>   <li> Subscriptions may be managed by the user and auto-renewal may be turned
>        off by going to the user's Account Settings after purchase </li>
>   <li> Any unused portion of a free trial period, if offered, will be forfeited
>        when the user purchases a subscription to that publication, where applicable </li>
>   </ul>
> </li>
> <li> A link to the terms of use </li>
> <li> A link to the privacy policy </li>
> </ul>
>
> Note: Adding the above information to the StoreKit modal alert is not
> sufficient; the information must also be displayed within the app itself,
> and it must be displayed clearly and conspicuously during the purchase
> flow without requiring additional action from the user, such as opening
> a link.
>
> Your app's **metadata** did not include:
>
> <ul>
> <li> The following information about the auto-renewable nature of the
>      subscription (same subitems as for app's binary) </li>
> <li> A link to the terms of use </li>
> <li> A privacy policy link in the Privacy Policy URL field of iTunes Connect </li>
> </ul>
>
> **Next Steps**
>
> To resolve this issue, please revise your app or its metadata to include
> the missing information.
>
> If the above information is in your app, please reply to this message in
> Resolution Center to provide details on where to locate it.

1. <https://stackoverflow.com/a/43651411/3632318>
2. <https://forums.developer.apple.com/thread/70917>
3. <http://captaindanko.blogspot.ru/2017/06/addressing-app-review-rejections-for.html>
