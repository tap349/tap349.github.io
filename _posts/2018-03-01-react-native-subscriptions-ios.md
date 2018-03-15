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

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>
2. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>

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
| `In-App Purchases` (left menu) → `In-App Purchases ⨁`

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
- `Subscription Prices ⨁`
  - `Currency` (combobox): `RUB - Russian Ruble`
  - `Price` (combobox): `149 р.`
- `Localizations ⨁` → `Russian`
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

- `Localizations ⨁` → `Russian`
  - `Subscription Group Display Name` (input): `Подписки`
  - `App Name Display Name`: [x] Use App Name

test subscription on real device
--------------------------------

1. [React Native - Running on Real Device]({% post_url 2018-03-05-react-native-running-on-real-device %})

it's required to run application on real device to make purchases:
the only action allowed from inside emulator is loading products.

subscriptions in test environment:

- have shortened periods (say, 1 month → 5 minutes)
- renew 5 times only (cancelled afterwards)
- cannot be managed (say, cancelled manually)

<https://forums.developer.apple.com/thread/14702>:

> the sandbox environment handles auto-renewing subscriptions in an automated
> fashion - that is subscription periods are shortened and renew only 5 times
> and not controlled through the subscription management screen.

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

prepare release with subscription
---------------------------------

### add new IAP when submitting new app version

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)
| `In-App Purchases (left menu)`

> Your first in-app purchase must be submitted with a new app version.
> Select it from the app’s In-App Purchases section and click Submit.
>
> Once your binary has been uploaded and your first in-app purchase has
> been submitted for review, additional in-app purchases can be submitted
> using the table below.
