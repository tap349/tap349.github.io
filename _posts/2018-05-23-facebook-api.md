---
layout: post
title: Facebook - API
date: 2018-05-23 15:06:01 +0300
access: public
comments: true
categories: [facebook]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

<dl>
  <dt>FD</dt>
  <dd>Facebook for Developers</dd>

  <dt>GAE</dt>
  <dd>Graph API Explorer</dd>
</dl>

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

## Facebook Login

1. [Permissions Reference - Facebook Login](https://developers.facebook.com/docs/facebook-login/permissions/v3.1)

## Graph API

1. <https://developers.facebook.com/docs/graph-api>

### get next bill date

1. <https://developers.facebook.com/docs/graph-api/changelog/version3.0>

you can get `active_billing_date_preference` and `next_bill_date` info only if
your token is generated for `Ads Manager` App (`App ID` field in ATD).

### get page conversations

1. <https://developers.facebook.com/docs/graph-api/reference/page/conversations/>
2. <https://developers.facebook.com/docs/graph-api/reference/v3.1/conversation>

first generate long-lived PAT with `read_page_mailboxes` permission - see `PAT`
section below.

this PAT can be now used to fetch page conversations:

```
UAT scopes: manage_pages,read_page_mailboxes
GET v3.1/me/conversations
```

### filter page conversations by date

> <https://developers.facebook.com/docs/graph-api/reference/v3.1/page/conversations>
>
> Time-based pagination is not available for the conversations endpoint.

filtering conversations by date is removed by Facebook:

> <https://developers.facebook.com/support/bugs/420363721670492>
>
> it appears as if we would have to delete conversations in order to prevent
> retrieving all conversations all the time.
>
> So if I want to get a message from January I have to paginate all over the
> object until January?

it's only possible to limit the number of conversations to fetch using `limit`
modifier.

## Marketing API

1. <https://developers.facebook.com/docs/marketing-apis>
2. [Business Manager](https://developers.facebook.com/docs/marketing-api/business-manager-api)

### get billing events (activities)

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-activity>

billing events:

- `add_funding_source` (payment method added - say, credit card)
- `funding_event_successful` (money added to balance)
- `ad_account_billing_charge` (ad account billed)

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages
GET v3.1/me/adaccounts?fields=id,account_id,name,activities{event_type,event_time,extra_data}
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
> certain amount known as your billing threshold and again on your monthly bill
> date for any leftover costs. This is how you'll pay if you use PayPal or most
> credit and debit cards to purchase ads.
>
> Manual payments: You'll add money to your account first, and then we'll deduct
> from that amount up to once a day as you run ads. This is how you'll pay if
> you use a manual payment method (like PayTM or Boleto Bancário) to purchase
> ads. With manual payments, you won't have a billing threshold.

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages
GET v3.1/me/adaccounts?fields=id,account_id,name,is_prepay_account
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
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages
GET v3.1/me/adaccounts?fields=id,account_id,name,funding_source_details
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

available balance in `funding_source_details` is decreased every day - on every
`ad_account_billing_charge` event, I guess. you can parse this string to predict
when user is close to running out of money.

### get campaign insights

1. <https://developers.facebook.com/docs/marketing-api/reference/ad-account/>

```
UAT scopes: email,public_profile,ads_management,ads_read,manage_pages
GET v3.1/me/adaccounts?fields=campaigns{insights{ctr,cpc,cpm,cpp}}
```

sample response:

```json
{
  "data": [
    {
      "campaigns": {
        "data": [
          {
            "insights": {
              "data": [
                {
                  "ctr": "1.201805",
                  "cpc": "19.106109",
                  "cpm": "229.61825",
                  "cpp": "326.688588",
                  "date_start": "2018-04-30",
                  "date_stop": "2018-05-29"
                }
              ]
            },
            "id": "<campaign_id_1>"
          },
          {
            "insights": {
              "data": [
                {
                  "ctr": "1.025733",
                  "cpc": "34.70614",
                  "cpm": "355.992442",
                  "cpp": "361.126323",
                  "date_start": "2018-04-30",
                  "date_stop": "2018-05-29"
                }
              ]
            },
            "id": "<campaign_id_2>"
          },
          {
            "id": "<campaign_id_3>"
          }
        ]
      },
      "id": "act_<account_id>"
    }
  ]
}
```

it turns out that `read_insights` permission is not required to read insights -
it must be sufficient to have `ads_read` permission only.

NOTE: insights are shown/available not for all campaigns - IDK why yet.

## user IDs

1. <https://developers.facebook.com/docs/messenger-platform/identity/id-matching>

- real user ID (UID)

  it can be used to open user's Facebook page (`https://facebook.com/<UID>`).

- app-scoped ID (ASID)

  > When a person uses Facebook Login on a website or a mobile app, an ID is
  > created for the specific Facebook app, which is called app-scoped ID.

  > <https://developers.facebook.com/docs/messenger-platform/send-messages#recipient_ids>
  >
  > Please note that user ID's from Facebook Login integrations are app-scoped
  > and will not work with the Messenger platform.

- page-scoped ID (PSID)

  > When a person interacts with a business via Messenger, an ID is created for
  > the specific Page associated with the bot in Messenger, which is called
  > Page-scoped ID.

## access tokens

1. <https://developers.facebook.com/docs/facebook-login/access-tokens>

- temporary token is called short-lived one
- permanent token is called long-lived one

short-lived token can be extended in ATD - this means it will be turned into
long-lived token (in fact it will be a brand new token then).

### UAT

> User access tokens are generally obtained via a login dialog and require a
> person to permit your app to obtain one.

### AAT

> It is generated using a pre-agreed secret between the app and Facebook

### PAT

#### generate long-lived PAT with messenger permissions in FD

1. [Facebook - Bot]({% post_url 2018-05-07-facebook-bot %})

| FB                                                           |
| ------------------------------------------------------------ |
| `PRODUCTS` (section in left menu) → `Messenger` → `Settings` |
| `Token Generation` (section) → `Select a Page` (combobox)    |

this PAT is long-lived and has only messenger permissions for selected page
(that is `pages_messaging_*` permissions).

**_UPDATE_**

new `KhLead` PAT generated this way now has not only messenger permissions but
`read_page_mailboxes` permission as well (as though it was saved after
generating PAT with `read_page_mailboxes` permission in GAE - see below).

#### generate long-lived PAT with arbitrary permissions in GAE

1. <https://stackoverflow.com/questions/17197970>

- get UAT with required permissions

  | GAE |
  | --- |

  - `Application` (combobox): `<MY_APP>`

  | GAE                                                   |
  | ----------------------------------------------------- |
  | `Get Token` (dropdown menu) → `Get User Access Token` |

  > Select Permissions

  - [x] `read_page_mailboxes`
  - `Get Access Token` (button)

- get short-lived PAT

  | GAE

  use UAT generated above to get short-lived PAT:

  ```
  UAT scopes: manage_pages,read_page_mailboxes
  GET v3.1/<PAGE_ID>?fields=access_token
  ```

  see [Facebook - Tips]({% post_url 2018-06-01-facebook-tips %}) on how to find
  page ID.

  => short-lived PAT is generated (expires in 1 hour) - now it's necessary to
  extend it and get a new long-lived PAT in ATD.

- get long-lived PAT

  | ATD

  paste short-lived PAT into ATD and click `Extend Access Token` button at the
  bottom of the page (you will be prompted to type your FB account password to
  confirm this action) - a new long-lived PAT will be generated that will never
  expire.
