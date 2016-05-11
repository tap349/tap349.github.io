---
layout: post
title: GA demystified
date: 2016-03-16 13:08:10 +0300
access: public
categories: [ga]
---

configuration of google analytics and some background information.

<!-- more -->

## [Google Analytics](https://analytics.google.com)

**Admin (3-column page: ACCOUNT - PROPERTY - VIEW)**

### ACCOUNT (1st column)

  **Account ID**

  - path: `Account Settings → Basic Settings → Account Id`
  - format: `\d+`
  - parametr in API: `accountId`
  - usage: API (e.g. `analytics.management.goals.*`)
  - pumba: not stored

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
  - pumba: not stored

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

## [API Manager](https://console.developers.google.com)

`Credentials (left sidebar) -> Credentials -> Credentials (tab)`

create OAuth 2.0 client IDs for each stage (Development, Staging, Production) -
they are stored in _secrets.yml_:

- Client ID - `oauth/google_analytics/app_id`
- Client secret - `oauth/google_analytics/app_secret`

client ID and client secret uniquely identify application.

each google API service (e.g. `Google Analytics API v3`)
defines one or more scopes that declare a set of operations permitted

general workflow
(<https://developers.google.com/api-client-library/ruby/start/get_started>):

- application requests access to user data (using `omniauth-google-oauth2` gem) -
  request includes one or more scopes
- application is authenticated with client ID and client secret
- user must authenticate himself - google account login screen is displayed
- user needs to approve scopes application is requesting -
  OAuth 2.0 consent screen is displayed to user
- when user grants access for application OAuth 2.0 authorization server
  provides application with refresh and access tokens -
  tokens are valid for the scopes requested only
- refresh token:
  - used to acquire new access token (client ID and client secret must be passed as well)
  - stored in database as `OauthToken`
  - never expires
- access token:
  - used to authorize API calls
  - stored in Rails cache only and fetched again when it expires
  - expires after period of time specified by authorization server
- application makes API call using access token only
