---
layout: post
title: GA demystified
date: 2016-03-16 13:08:10 +0300
access: public
categories: [ga]
---

configuration of [Google Analytics](https://analytics.google.com)
and some background information.

<!-- more -->

* TOC
{:toc}

## Google Analytics IDs

**Admin (3-column page: ACCOUNT - PROPERTY - VIEW)**

### ACCOUNT (1st column)

  **Account ID**

  - path: `Account Settings → Basic Settings → Account Id`
  - format: `\d+`
  - parametr in API: `accountId`
  - usage: API (e.g. `analytics.management.goals.*`)
  - pumba: `additional_ids['account_id']` in `Google::Analytics::Counter`

### PROPERTY (2nd column)

  **Tracking ID**

  - path (displayed in both places):
    - `Property Settings → Basic Settings → Tracking Id`
    - `Tracking Info → Tracking Code → Tracking ID`
  - format: `UA-XXXX-1` where `XXXX` - Account ID (see above)
  - parameter in API: `webPropertyId`
  - usage:
    - API (e.g. `analytics.management.goals.*`)
    - in Universal Analytics tracking code
      (GA JS script embedded on each tracked page)
  - pumba: `additional_ids['web_property_id']` in `Google::Analytics::Counter`

### VIEW (3rd column)

  **View ID (Profile ID)**

  - path: `View Settings → Basic Settings → View ID`
  - format: `\d+`
  - parameter in API: `profileId`
  - usage: API (e.g. `analytics.management.goals.*`)
  - pumba: not stored

  **Table ID**

  - path: not displayed in UI
  - format: `ga:XXXX` where `XXXX` - View ID
  - parameter in API: `ids` (in `analytics.data.ga.get`)
  - usage: API (e.g. `analytics.data.ga.get`)
  - pumba: `counter_id` in `Google::Analytics::Counter`

## Google Analytics terms

### OAuth 2.0 client ID

client ID:

- is used to authenticate application via OAuth 2.0
- has client secret (aka API key)
- is created in [API Manager](https://console.developers.google.com):

  `Credentials (left sidebar) -> Credentials -> Credentials (tab)`

pumba client IDs for all 3 stages (Development, Staging, Production)
were created in `finga***88@gmail.com` account!

client ID for Development is stored in _secrets.yml_:

- client ID - `oauth/google_analytics/app_id`
- client secret - `oauth/google_analytics/app_secret`

client IDs for Staging and Production are stored in chef.

### API methods

all Google Analytics API methods for `Google Analytics API v3` service
are listed [here](https://developers.google.com/apis-explorer/#p/analytics/v3/).

API methods currently used in pumba:

- `analytics.data.ga.get`
- `analytics.data.mcf.get`
- `analytics.management.accounts.list`
- `analytics.management.profiles.*`
- `analytics.management.goals.*`

### scope

each Google API service (e.g. `Google Analytics API v3`)
defines one or more scopes that declare a set of operations permitted.

## using [APIs Explorer](https://developers.google.com/apis-explorer/#p/)

APIs Explorer supplies its own default credentials
(client ID and client secret) when executing queries.
or else it's possible to supply custom credentials:

`Settings icon in upper right corner -> Set API key / OAuth 2.0 Client ID -> Custom credentials`

as a rule using default credentials must be enough as it's all the same
it's necessary to grant access to application when OAuth 2.0 consent screen
is displayed - it doesn't matter whether this is pumba application or not.

to execute query it's necessary to get access to user data -
for this to happen user must authenticate himself (log in) and approve
[scopes](#scope) application (either default one or pumba) is requesting.

**NOTE**: beware of the situation when user is already logged (either in Chrome
          browser or on website) and this is wrong user - log out in this case.

### using APIs Explorer in pumba

in pumba user counters are created in:

- `re***2015@gmail.com` account - primary counters (for visits/visitors statistics)
- `re***.counter@gmail.com` account - MCF counters (for assisted conversions statistics)

consequently it's necessary to log in one of these accounts
(when Google account login screen is displayed) to get access to:

- information related to user counters (e.g. goals)
- and counter statistics itself

## API usage workflow

<https://developers.google.com/api-client-library/ruby/start/get_started>

- application requests access to user data (using `omniauth-google-oauth2` gem) -
  request includes one or more scopes
- application is authenticated with client ID and client secret
- user must authenticate himself - Google account login screen is displayed
- user needs to approve scopes application is requesting -
  OAuth 2.0 consent screen is displayed to user
- when user grants access for application OAuth 2.0 authorization server
  provides application with refresh and access tokens -
  tokens are valid for the scopes requested only
- refresh token:
  - used to acquire new access token (client ID and client secret must be passed as well)
  - stored in database as `token` in `Google::Analytics::Token`
  - never expires
- access token:
  - used to authorize API calls
  - stored in Rails cache only and fetched again when it expires
  - expires after period of time specified by authorization server
- application makes API call using access token only
