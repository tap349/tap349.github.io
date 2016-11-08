---
layout: post
title: GA demystified
date: 2016-03-16 13:08:10 +0300
access: public
categories: [ga]
---

information about [Google Analytics](https://analytics.google.com) and its API.

<!-- more -->

* TOC
{:toc}

## Google Analytics (account, tracking and view IDs)

<https://analytics.google.com/analytics/web/#management/Settings>

Administration page has 3 columns: ACCOUNT - PROPERTY - VIEW.

### ACCOUNT (1st column)

  **account ID**

  - path: `Account Settings → Basic Settings → Account Id`
  - format: `\d+`
  - parameter in API: `accountId`
  - usage: API (e.g. `analytics.management.goals.*`)
  - pumba: `additional_ids['account_id']` in `Google::Analytics::Counter`

### PROPERTY (2nd column)

  **tracking ID**

  - path (displayed in both places):
    - `Property Settings → Basic Settings → Tracking Id`
    - `Tracking Info → Tracking Code → Tracking ID`
  - format: `UA-XXXX-Y` where `XXXX` - account ID (see above),
    `Y` - ordinal number of tracking ID (1, 2, etc.)
  - parameter in API: `webPropertyId`
  - usage:
    - API (e.g. `analytics.management.goals.*`)
    - in Universal Analytics tracking code
      (GA JS script embedded on each tracked page)
  - pumba: `additional_ids['web_property_id']` in `Google::Analytics::Counter`

### VIEW (3rd column)

  **view ID (profile ID)**

  - path: `View Settings → Basic Settings → View ID`
  - format: `\d+`
  - parameter in API: `profileId`
  - usage: API (e.g. `analytics.management.goals.*`)
  - pumba: not stored

  **table ID**

  - path: not displayed in UI
  - format: `ga:XXXX` where `XXXX` - View ID
  - parameter in API: `ids` (in `analytics.data.ga.get`)
  - usage: API (e.g. `analytics.data.ga.get`)
  - pumba: `counter_id` in `Google::Analytics::Counter`

## API Manager (client IDs)

<https://console.developers.google.com>

OAuth 2.0 client IDs:

- are used to identify your application (project)
- have corresponding client secrets (don't confuse with API keys!)
- are managed in API Manager:

  `Credentials (left sidebar) -> Credentials -> Credentials (tab)`

API keys:

- are used to identify your application (project)
- don't request user consent unlike OAuth 2.0 client ID
- are managed in API Manager:

  `Credentials (left sidebar) -> Credentials -> Credentials (tab)`

### pumba client IDs

**NOTE**: select pumba project (`Re***`) in top left menu beforehand
          (here you can also switch between projects targeted either
          for GA or GAW - this grouping is our own, not Google's one).

pumba client IDs for all 3 stages (Development, Staging, Production)
were created in `finga***88@gmail.com` account!

client ID for Development is stored in _secrets.yml_:

- client ID - `oauth/google_analytics/app_id`
- client secret - `oauth/google_analytics/app_secret`

client IDs for Staging and Production are stored in chef.

### pumba API keys

API keys are not used in pumba.

## APIs Explorer

<https://developers.google.com/apis-explorer>

<https://ga-dev-tools.appspot.com/query-explorer> - Query Explorer
(almost the same as APIs Explorer but somewhat easier to use since it requires
authorization and prefetches existing values - accounts, properties, views)

APIs Explorer allows to make API requests to different API services
(optionally with OAuth 2.0 authorization) and exposes the same methods
as those available in client libraries.

### API services

all services available in APIs Explorer are listed
[here](https://developers.google.com/apis-explorer) -
select `Google Analytics API v3` service.

### API methods vs HTTP requests

all methods for `Google Analytics API v3` service are listed
[here](https://developers.google.com/apis-explorer/#p/analytics/v3/).

these methods are exposed in client libraries for different languages
and have language specific interfaces to set query parameters.

API can also be queried as REST-ful endpoint by making HTTP request:

e.g. calling `analytics.data.ga.get` method with `ids=ga:12345` argument
(either in client library or APIs Explorer) is equivalent to making
`https://www.googleapis.com/analytics/v3/data/ga?ids=ga:12345` HTTP request
(in fact this method has more required arguments than shown here).

when making HTTP request arguments can be supplied as:

- parameters in URL query string (`...?ids=ga:12345`)
- segments in URL path (`.../accounts/accountId/...`)

methods currently used in pumba:

- `analytics.data.ga.get` (visits, visitors)
- `analytics.data.mcf.get` (assisted conversions)
- `analytics.management.accounts.list`
- `analytics.management.profiles.*`
- `analytics.management.goals.*` (goals, goal completions)

**NOTE**: in pumba we don't use client libraries and make HTTP requests directly -
          methods are listed above for the sake of clarity only.

### scopes

scopes are used to grant an application different levels of access to data
on behalf of the end user. each API may declare one or more scopes.

Google Analytics API declares a set of scopes:

- `https://www.googleapis.com/auth/analytics`<br>
  (view and manage your Google Analytics data)
- `https://www.googleapis.com/auth/analytics.edit`<br>
  (edit Google Analytics management entities)
- `https://www.googleapis.com/auth/analytics.manage.users`<br>
  (manage Google Analytics Account users by email address)
- `https://www.googleapis.com/auth/analytics.manage.users.readonly`<br>
  (view Google Analytics user permissions)
- `https://www.googleapis.com/auth/analytics.provision`<br>
  (create a new Google Analytics account along with its default property and view)
- `https://www.googleapis.com/auth/analytics.readonly`<br>
  (view your Google Analytics data)

to get Google Analytics data it's necessary to approve at least the last scope.

### application and user authentication

APIs Explorer provides its own default credentials
(client ID and client secret) when executing queries.
or else it's possible to provide custom credentials:

`Settings icon in upper right corner -> Set API key / OAuth 2.0 Client ID -> Custom credentials`

as a rule using default credentials must be sufficient as it's all the same
it's necessary to grant access to application when OAuth 2.0 consent screen
is displayed - it doesn't matter whether this is pumba application or not.

to execute query it's necessary to get access to user data -
for this to happen user must authenticate himself (log in) and approve
scopes application (either default one or pumba) is requesting.

**NOTE**: beware of the situation when user is already logged in (either in Chrome
          browser or via website) and this is a wrong user - log out in this case.

### pumba accounts

in pumba user counters are accessed via these pumba accounts:

- `re***2015@gmail.com` account - primary counters (for visits/visitors statistics)
- `re***.counter@gmail.com` account - MCF counters (for assisted conversions statistics)

primary counters are created by users in their accounts and just shared
with our `re***2015@gmail.com` account.
MCF counters are created by ourselves in `re***.counter@gmail.com` account
(manually - not automatically in pumba).

consequently it's necessary to log in one of pumba accounts
(when Google account login screen is displayed) to get access to:

- information related to user counters (e.g. goals)
- and counter statistics itself

## acquiring refresh and access tokens for application

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
  - stored in database as `token` in `OauthToken` (`Google::Analytics::Token`, etc.)
  - never expires
- access token:
  - used to authorize API calls
  - stored in Rails cache only and fetched again when it expires
  - expires after period of time specified by authorization server
- application makes API call using access token only
