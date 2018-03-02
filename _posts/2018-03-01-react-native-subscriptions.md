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

> `iTunes Connect` → `Features` (tab) → `In-App Purchases` (left menu)

- click `⨁` link to start IAP creation wizard:

  > `Select the in-app purchase you want to create.`

  [x] `Auto-Renewable Subscription`

  > `Create Auto-Renewable Subscription`

  fill `Reference Name` and `Product ID` fields:
  say, `No Ads Monthly` and `<bundle_id>.sub.noads.monthly` where
  `<bundle_id>` is the value of `PRODUCT_BUNDLE_IDENTIFIER` property
  in _project.pbxproj_ (`com.iceperk.iceperkapp` for our application).

  product ID must be unique - even after deleting subscription
  its product ID cannot be reused for new subscriptions.

  > `Create Subscription Group`

  [x] `Create New Subscription Group`

  fill `Subscription Group Reference Name` field:
  say, `No Ads` (must be more general than subscription reference name).

- make sure new subscription is cleared for sale

  [x] `Cleared for Sale`

- TODO: subscription, localization (Скрытие рекламы на месяц),
  no localization for subscription group

- add localization in at least one language for subscription group
- add localization in at least one language for subscription
