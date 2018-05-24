---
layout: post
title: Facebook - API
date: 2018-05-23 15:06:01 +0300
access: private
comments: true
categories: [facebook]
---

<!-- more -->

* TOC
{:toc}
<hr>

- ATD - Access Token Debugger

Facebook Login
--------------

1. [Permissions Reference - Facebook Login](https://developers.facebook.com/docs/facebook-login/permissions/v3.0)

Graph API
---------

1. <https://developers.facebook.com/docs/graph-api>

### next bill date

1. [Version 3.0](https://developers.facebook.com/docs/graph-api/changelog/version3.0)

you can get `active_billing_date_preference` and `next_bill_date` info only
if your token is generated for `Ads Manager` App (`App ID` field in ATD).

Marketing API
-------------

1. <https://developers.facebook.com/docs/marketing-apis>
2. [Business Manager](https://developers.facebook.com/docs/marketing-api/business-manager-api)

### get ad_account_billing_charge events (activities)

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-activity>

this event is sent when ad account is billed.

- client access token scopes

  ```
  email,public_profile,ads_management,ads_read,manage_pages,read_insights
  ```

- request

  ```
  GET v3.0/me/adaccounts?fields=id,account_id,name,activities{event_type,event_time,extra_data}
  ```

- sample response

  ```json
  {
    "data": [
      {
        "id": "act_123",
        "account_id": "123",
        "name": "John Doe",
        "activities": {
          "data": [
            {
              "event_type": "ad_account_billing_charge",
              "event_time": "2018-05-21T13:34:11+0000",
              "extra_data": "{\"currency\":\"USD\",\"new_value\":100,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
            },
            {
              "event_type": "ad_account_billing_charge",
              "event_time": "2018-05-20T13:37:46+0000",
              "extra_data": "{\"currency\":\"USD\",\"new_value\":933,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
            },
            {
              "event_type": "ad_account_billing_charge",
              "event_time": "2018-05-19T13:37:11+0000",
              "extra_data": "{\"currency\":\"USD\",\"new_value\":1047,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
            },
            {
              "event_type": "ad_account_billing_charge",
              "event_time": "2018-05-18T14:02:07+0000",
              "extra_data": "{\"currency\":\"USD\",\"new_value\":728,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
            },
            {
              "event_type": "ad_account_billing_charge",
              "event_time": "2018-05-17T14:19:58+0000",
              "extra_data": "{\"currency\":\"USD\",\"new_value\":366,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
            }
          ]
        }
      }
    ]
  }
  ```

### check if account is prepaid or not

1. <https://www.facebook.com/business/help/105373712886516>

> <https://www.facebook.com/business/help/105373712886516>
>
> There are two main payment settings for Facebook ads:
>
> Automatic payments: We'll automatically charge you whenever you spend a
> certain amount known as your billing threshold and again on your monthly
> bill date for any leftover costs. This is how you'll pay if you use PayPal
> or most credit and debit cards to purchase ads.
>
> Manual payments: You'll add money to your account first, and then we'll
> deduct from that amount up to once a day as you run ads. This is how you'll
> pay if you use a manual payment method (like PayTM or Boleto Bancário) to
> purchase ads. With manual payments, you won't have a billing threshold.

- client access token scopes

  ```
  email,public_profile,ads_management,ads_read,manage_pages,read_insights
  ```

- request

  ```
  GET v3.0/me/adaccounts?fields=id,account_id,name,is_prepay_account
  ```

- sample response

  ```json
  {
    "data": [
      {
        "id": "act_123",
        "account_id": "123",
        "name": "John Doe",
        "is_prepay_account": true
      }
    ]
  }
  ```

### get payment method (funding source) and available balance

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-account/>

- client access token scopes

  ```
  email,public_profile,ads_management,ads_read,manage_pages,read_insights
  ```

- request

  ```
  GET v3.0/me/adaccounts?fields=id,account_id,name,funding_source_details
  ```

- sample response

  ```json
  {
    "data": [
      {
        "id": "act_123",
        "account_id": "123",
        "name": "John Doe",
        "funding_source_details": {
          "id": "345",
          "display_string": "Available Balance ($24.52 USD)",
          "type": 20
        }
      }
    ]
  }
  ```
