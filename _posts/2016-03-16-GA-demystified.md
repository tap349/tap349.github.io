---
layout: post
title: GA demystified
date: 2016-03-16 13:08:10 +0300
access: public
categories: [ga]
---

configuration of google analytics and some background information.

<!-- more -->

### <http://analytics.google.com>

- **Tracking ID** (`UA-XXXX-1`)

  `Admin -> Property -> Tracking Info -> Tracking Code`

  - `XXXX` - account id
  - tracking id is used in Universal Analytics tracking code
    (GA js script embedded on each tracked page)
  - currently not stored in Pumba at all

- **View (Profile) ID** (`XXXX`)

  `Admin -> View -> View Settings`

  - `ga:XXXX` - unique table id for retrieving analytics data
  - unique table id is used as `ids` parameter in GA API query
  - stored in `Google::Analytics::Counter` in Pumba (as `ga:XXXX`)

### <https://console.developers.google.com>

`Enable APIs and get credentials like keys -> Credentials -> Credentials`

generate OAuth 2.0 client IDs for each stage (Development, Staging, Production) -
they are stored in _secrets.yml_:

- Client ID - `oauth/google_analytics/app_id`
- Client secret - `oauth/google_analytics/app_secret`

client ID and client secret uniquely identify application

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
