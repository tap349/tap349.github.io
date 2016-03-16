---
layout: post
title: GA demystified
date: 2016-03-16 13:08:10 +0300
access: public
categories: [ga]
---

## http://analytics.google.com

- **Tracking ID** (UA-\d{8}-1)

  `Admin -> Property -> Tracking Info -> Tracking Code`

  used in Universal Analytics tracking code (GA script embedded on each tracked page)

  NOTE: currently not stored in Pumba at all

- **View ID**

  `Admin -> View -> View Settings`

  used in GA API as counter id with `ga:` prefix

  NOTE: stored in `AnalyticsCounter` in Pumba as `ga:\d{8}`

## https://console.developers.google.com

`Enable APIs and get credentials like keys -> Credentials -> Credentials`

generate OAuth 2.0 client IDs for each stage (Development, Staging, Production) -
they are stored in _secrets.yml_:

Client ID - `oauth/google_analytics/app_id`
Client secret - `oauth/google_analytics/app_secret`

when app needs access to GA counter statistics:

- it goes to GA site
- user is asked to sign in using his google account if necessary
- OAuth consent screen is displayed with available scopes that can be granted
  to your app by user -
  different google API services require different scopes to be granted
- if user grants requested permissions (scopes) GA site returns OAuth token
  that is stored in database (`OauthToken`) and can be used later
  to perform requests to GA API.

OAuth token allows to access user's data provided by any GA service
(e.g. `Google Analytics API v3`) for which granted set of scopes is sufficient.
your app is identified by client id and secret - these parameters along with
refresh token are passed in every request. refresh token is acquired based on
OAuth token and has limited expiration period.
