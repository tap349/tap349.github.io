---
layout: post
title: Facebook - API
date: 2018-05-23 15:06:01 +0300
access: public
comments: true
categories: [facebook]
---

<!-- more -->

* TOC
{:toc}
<hr>

- user IDs

  <dl>
    <dt>UID</dt>
    <dd>real user ID</dd>

    <dt>ASID</dt>
    <dd>app-scoped ID</dd>

    <dt>PSID</dt>
    <dd>page-scoped ID</dd>
  </dl>

- access tokens

  <dl>
    <dt>AT</dt>
    <dd>access token</dd>

    <dt>UAT</dt>
    <dd>user access token</dd>

    <dt>AAT</dt>
    <dd>app access token</dd>

    <dt>PAT</dt>
    <dd>page access token</dd>

    <dt>ATD</dt>
    <dd>Access Token Debugger</dd>
  </dl>

<hr>

Facebook Login
--------------

1. [Permissions Reference - Facebook Login](https://developers.facebook.com/docs/facebook-login/permissions/v3.0)

Graph API
---------

1. <https://developers.facebook.com/docs/graph-api>

### get next bill date

1. <https://developers.facebook.com/docs/graph-api/changelog/version3.0>

you can get `active_billing_date_preference` and `next_bill_date` info only
if your token is generated for `Ads Manager` App (`App ID` field in ATD).

Marketing API
-------------

1. <https://developers.facebook.com/docs/marketing-apis>
2. [Business Manager](https://developers.facebook.com/docs/marketing-api/business-manager-api)

### get billing events (activities)

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-activity>

billing events:

- `add_funding_source` (payment method added - say, credit card)
- `funding_event_successful` (money added to balance)
- `ad_account_billing_charge` (ad account billed)

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages,read_insights
GET v3.0/me/adaccounts?fields=id,account_id,name,activities{event_type,event_time,extra_data}
```

sample response:

```json
{
  "data": [
    {
      "id": "act_123",
      "account_id": "123",
      "name": "RS",
      "activities": {
        "data": [
          {
            "event_type": "funding_event_successful",
            "event_time": "2018-05-25T04:40:16+0000",
            "extra_data": "{\"action\":67,\"amount\":3000,\"currency\":\"USD\",\"fee\":0,\"network_id\":0,\"provider_amount\":3000,\"type\":\"\"}"
          },
          {
            "event_type": "add_funding_source",
            "event_time": "2018-05-25T04:40:16+0000",
            "extra_data": "{\"payment_method\":{\"__html\":\"Credit Card\"}}"
          },
          {
            "event_type": "ad_account_billing_charge",
            "event_time": "2018-05-24T14:15:03+0000",
            "extra_data": "{\"currency\":\"USD\",\"new_value\":611,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
          },
          {
            "event_type": "funding_event_successful",
            "event_time": "2018-05-24T01:44:54+0000",
            "extra_data": "{\"action\":67,\"amount\":2000,\"currency\":\"USD\",\"fee\":0,\"network_id\":0,\"provider_amount\":2000,\"type\":\"\"}"
          },
          {
            "event_type": "add_funding_source",
            "event_time": "2018-05-24T01:44:54+0000",
            "extra_data": "{\"payment_method\":{\"__html\":\"Credit Card\"}}"
          },
          {
            "event_type": "ad_account_billing_charge",
            "event_time": "2018-05-23T14:44:39+0000",
            "extra_data": "{\"currency\":\"USD\",\"new_value\":984,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
          },
          {
            "event_type": "ad_account_billing_charge",
            "event_time": "2018-05-22T14:44:04+0000",
            "extra_data": "{\"currency\":\"USD\",\"new_value\":1804,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
          },
          {
            "event_type": "ad_account_billing_charge",
            "event_time": "2018-05-21T13:34:11+0000",
            "extra_data": "{\"currency\":\"USD\",\"new_value\":100,\"transaction_id\":\"999\",\"action\":67,\"type\":\"payment_amount\"}"
          },
          {
            "event_type": "funding_event_successful",
            "event_time": "2018-05-21T05:54:26+0000",
            "extra_data": "{\"action\":67,\"amount\":3000,\"currency\":\"USD\",\"fee\":0,\"network_id\":0,\"provider_amount\":3000,\"type\":\"\"}"
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

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages,read_insights
GET v3.0/me/adaccounts?fields=id,account_id,name,is_prepay_account
```

sample response:

```json
{
  "data": [
    {
      "id": "act_123",
      "account_id": "123",
      "name": "RS",
      "is_prepay_account": true
    }
  ]
}
```

### get payment method (funding source) and available balance

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-account/>

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages,read_insights
GET v3.0/me/adaccounts?fields=id,account_id,name,funding_source_details
```

sample response:

```json
{
  "data": [
    {
      "id": "act_123",
      "account_id": "123",
      "name": "RS",
      "funding_source_details": {
        "id": "345",
        "display_string": "Available Balance ($24.52 USD)",
        "type": 20
      }
    }
  ]
}
```

available balance in `funding_source_details` is decreased every day - on
every `ad_account_billing_charge` event, I guess. you can parse this string
to predict when user is close to running out of money.

user IDs
--------

1. <https://developers.facebook.com/docs/messenger-platform/identity/id-matching>

- real user ID (UID)

  it can be used to open user's Facebook page (`https://facebook.com/<UID>`).

- app-scoped ID (ASID)

  > When a person uses Facebook Login on a website or a mobile app, an ID is
  > created for the specific Facebook app, which is called app-scoped ID.

- page-scoped ID (PSID)

  > When a person interacts with a business via Messenger, an ID is created
  > for the specific Page associated with the bot in Messenger, which is called
  > Page-scoped ID.

access tokens
-------------

1. <https://developers.facebook.com/docs/facebook-login/access-tokens>

- user access token (UAT)

  > User access tokens are generally obtained via a login dialog and
  > require a person to permit your app to obtain one.

- app access token (AAT)

  > It is generated using a pre-agreed secret between the app and Facebook

- page access token (PAT)

  > To obtain a page access token you need to start by obtaining a user
  > access token and asking for the manage_pages permission. Once you have
  > the user access token you then get the page access token via the Graph API.

  obtain PAT:

  ```
  UAT scopes: manage_pages
  GET v3.0/<page_id>?fields=access_token
  ```

  PATs are short-lived (expire in 1 hour).

account linking
---------------

1. <https://developers.facebook.com/docs/messenger-platform/identity/account-linking>
2. <http://blog.99array.com/2017/05/28/facebook-account-linking/>

> <https://medium.com/@philippholly/bbe632c578ca>
>
> AFAIK yes, you can open a webview with your own hosted website where you grab
> the messenger user id, and tell the user to click on “login with facebook”.
> Then you get the APP ID and can save a relation in your database with the
> messenger user id.
