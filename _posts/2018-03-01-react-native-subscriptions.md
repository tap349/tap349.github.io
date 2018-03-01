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

### create in-app purchase (IAP)

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>

- open `iTunes Connect` in browser
  - go to `Features` tab
  - go to `In-App Purchases` (left menu)
  - click `⨁` link to start IAP creation wizard:

    > Select the in-app purchase you want to create.

    [x] `Auto-Renewable Subscription`

    > Create Auto-Renewable Subscription

    fill `Reference Name` and `Product ID` fields

    e.g.: `Subscription to Remove Ads` and `<tld>.<site>.<app>.removeads`

    product ID must be unique - even after deleting subscription
    its product ID cannot be used for new subscriptions.

    > Create Subscription Group

    [x] `Create New Subscription Group`

    e.g.: `Subscriptions to Remove Ads`

  - add localization in at least one language for subscription group
  - add localization in at least one language for subscription
