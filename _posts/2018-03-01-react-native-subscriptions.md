---
layout: post
title: React Native - Subscriptions
date: 2018-03-01 18:40:21 +0300
access: private
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

iOS
---

### create in-app purchase (IAP)

| iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>
2. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>

- create new IAP: | `In-App Purchases (left menu)` → `In-App Purchases ⨁`

  | `Select the in-app purchase you want to create.`

  - [x] `Auto-Renewable Subscription`

  | `Create Auto-Renewable Subscription`

  - `Reference Name`: `No Ads Monthly`
  - `Product ID`: `<bundle_id>.sub.noads.monthly`

    `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
    in _project.pbxproj_ - `com.iceperk.iceperkapp` for our application.

    product ID must be unique - even after deleting subscription
    its product ID cannot be reused for new subscriptions.

  | `Create Subscription Group`

  - [x] `Create New Subscription Group`
    - `Subscription Group Reference Name`: `No Ads`

      must be more general than subscription reference name.

- configure new IAP: | `In-App Purchases (left menu)` → `<new_subscription>`

  - [x] `Cleared for Sale`
  - `Subscription Duration` → `1 Month`
  - `Subscription Prices ⨁`
    - `Currency`: `RUB - Russian Ruble`
    - `Price`: `149 р.`
  - `Localizations ⨁` → `Russian`
    - `Subscription Display Name`: `Скрытие рекламы на месяц`
    - `Description`: don't add so far (maybe not required)

- don't add localization for subscription group (maybe not required)

| iTunes Connect: `My Apps` → `Users and Roles`

1. <https://support.magplus.com/hc/en-us/articles/203809008-iOS-How-to-Test-In-App-Purchases-in-Your-App>
2. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>

- create sandbox tester (test user): | `Sandbox Testers` (tab) → `Testers ⨁`

  new sandbox tester shouldn't have an existing Apple ID (for some reason):
  I got `That email address is already associated with an existing Apple ID`
  error when I tried to add myself as a sandbox tester - Apple ID is created
  for each new sandox tester right after it's added here.

  don't forget to verify new test account (follow the link sent to specified
  email):

  > You must validate your e-mail, or any purchases you make will silently fail.

  **UPDATE**

  I couldn't verify account of a new sandbox tester - after clicking the
  link email verification page kept on loading forever with an activity
  indicator in the middle of the screen and JS errors in Chrome Console:

  ```
  bundle.js:46270 Refused to create a worker from 'blob:https://id.apple.com/...'
  because it violates the following Content Security Policy directive...
  ```

  still I'll try to use this account for testing subscriptions - they say
  on the Internet it might come off even though it's not verified.
