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
  - stored in `AnalyticsCounter` in Pumba 

### <https://console.developers.google.com>

`Enable APIs and get credentials like keys -> Credentials -> Credentials`

generate OAuth 2.0 client IDs for each stage (Development, Staging, Production) -
they are stored in _secrets.yml_:

- Client ID - `oauth/google_analytics/app_id`
- Client secret - `oauth/google_analytics/app_secret`

when app needs access to GA counter statistics:

- it goes to GA site
- user is asked to sign in using his google account if necessary
- OAuth consent screen is displayed with scopes required for selected API
  service that must be granted to app by user to use this service -
  different google API services require different scopes to be granted
- if user grants requested permissions (scopes) GA site returns OAuth token
  that is stored in database (`OauthToken`) and can be used later
  to perform requests to GA API

OAuth token allows to access user's data provided by any GA service
(e.g. `Google Analytics API v3`) for which granted set of scopes is sufficient.
app is identified by client id and client secret - these parameters along with
refresh token are passed in every request. refresh token is acquired using
OAuth token and has limited expiration period during which it can be reused.
