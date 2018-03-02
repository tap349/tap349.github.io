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

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>
2. <https://www.raywenderlich.com/154737/app-purchases-auto-renewable-subscriptions-tutorial>

> iTunes Connect: `Features` (tab) → `In-App Purchases` (left menu)

- click `⨁` link to start IAP creation wizard:

  > `Select the in-app purchase you want to create.`

  - [x] `Auto-Renewable Subscription`

  > `Create Auto-Renewable Subscription`

  - `Reference Name`: `No Ads Monthly`
  - `Product ID`: `<bundle_id>.sub.noads.monthly`

    `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
    in _project.pbxproj_ - `com.iceperk.iceperkapp` for our application.

    product ID must be unique - even after deleting subscription
    its product ID cannot be reused for new subscriptions.

  > `Create Subscription Group`

  - [x] `Create New Subscription Group`
    - `Subscription Group Reference Name`: `No Ads`

      must be more general than subscription reference name.

- [x] `Cleared for Sale`
- `Subscription Duration` → `1 Month`
- `Localizations` → `Russian`
  - `Subscription Display Name`: `Скрытие рекламы на месяц`
  - `Description`: don't add so far (maybe it's not required)
- don't add localization for subscription group so far
