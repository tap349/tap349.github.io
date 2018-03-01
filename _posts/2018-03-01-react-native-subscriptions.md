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

### create new in-app purchase (IAP)

1. <https://distriqt.github.io/ANE-InAppBilling/s.Apple%20In-App%20Purchases>

- open `iTunes Connect` in browser
  - go to `Features` tab
  - go to `In-App Purchases` (left menu)
  - click `⨁` link to start IAP creation wizard:

    > Select the in-app purchase you want to create.

    [x] select `Auto-Renewable Subscription`

    > Create Auto-Renewable Subscription

    fill `Reference Name` and `Product ID` fields
    (say, `Subscription to Remove Ads` and `<tld>.<site>.<app>.removeads`)

    product id must be unique - even after deleting subscription
    you cannot use its product ids for new subscriptions.

    > Create Subscription Group

    create new subscription group (say, `Subscriptions to Remove Ads`)

  - add localization in at least one language for subscription group
  - add localization in at least one language for subscription
