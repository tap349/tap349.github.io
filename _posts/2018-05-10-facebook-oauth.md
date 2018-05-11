---
layout: post
title: Facebook - OAuth
date: 2018-05-10 13:06:44 +0300
access: private
comments: true
categories: [facebook]
---

<!-- more -->

* TOC
{:toc}
<hr>

- FD - Facebook for Developers

whitelist redirect URI
----------------------

1. <https://help.sharetribe.com/managing-your-marketplace/social-media/how-to-solve-the-url-blocked-this-redirect-failed-because-facebook-login-error>
2. <https://stackoverflow.com/questions/2459728/how-to-test-facebook-connect-locally>

it's necessary to whitelist redirect URI - all other redirect URIs will be
blocked by Facebook:

> URL Blocked: This redirect failed because the redirect URI is not whitelisted
> in the app’s Client OAuth Settings. Make sure Client and Web OAuth Login are
> on and add all your app domains as Valid OAuth Redirect URIs.

### add local domain to /etc/hosts

_/etc/hosts_:

```diff
+ 127.0.0.1 sith.local
```

or else it's possible to whitelist `localhost` as a valid redirect URI in FD
without using any custom local domains.

<https://stackoverflow.com/a/5626979/3632318>:

> Facebook does not "connect" back to your server. Their JS does. And the JS
> runs in the context of your browser. Which knows where "localhost" points to.

### whitelist local domain in FD

<https://wp-native-articles.com/blog/news/how-to-fix-facebook-apps-error-cant-load-url-domain-url-isnt-included-apps-domains/>:

> Any new Facebook Login Apps create AFTER the beginning of March 2018 now
> have Use Strict Mode for Redirect URIs and Enforce HTTPS enabled by default
> and can no longer be disabled.
>
> ...it means that you now have to put the exact return URL into the Valid
> OAuth Redirect URIs input. Previously, with strict mode disabled, you could
> just put your domain name in and that would be enough.

=> you must use exact redirect URI - you can't just enter domain name as a
valid redirect URI (say, `http://sith.local`) or else you'll get this error:

> Can't Load URL: The domain of this URL isn't included in the app's domains.
> To be able to load this URL, add all domains and subdomains of your app to
> the App Domains field in your app settings.

| FD: `PRODUCTS` (section in left menu) → `Facebook Login` → `Settings`
| `Client OAuth Settings` (section)

- `Valid OAuth Redirect URIs` (input): add `http://sith.local:4000/auth/facebook/callback`

request user for permissions
----------------------------

1. <https://developers.facebook.com/docs/marketing-api/access#manually-getting-access-tokens>
2. <https://developers.gigya.com/display/GD/Facebook+Login+Permissions#FacebookLoginPermissions-AvailablePermissions>

NOTE: scopes = permissions.

first user clicks the link:

```
http://sith.local:4000/auth/facebook?scope=email,public_profile,ads_management,ads_read
```

then OAuth library redirects to `https://www.facebook.com/dialog/oauth`
with additional query params including `client_id` (the request is made
on behalf of your app).

there're 2 possible outcomes from here on depending on user choice -
either app or business integration can be added to his Facebook account:

- app with data access

  | Facebook: `▼` (top right menu) → `Settings`
  | `Apps and Websites` (left menu) → `Active` (tab)

  added app will have `email,public_profile` scopes.

- business integration with data and account access

  | Facebook: `▼` (top right menu) → `Settings`
  | `Business Integrations` (left menu) → `Active` (tab)

  added business integration will have all permissions
  (`email,public_profile,ads_management,ads_read`).

corresponding permissions might be revoked by removing either app or
business integration from user's Facebook account.

any successful response from Facebook contains, inter alia, access token
(JWT token) but its scopes may different based on user action. if access
token with requested scopes has been created before, Facebook won't prompt
user next time request is made but will return a new token with the same
scopes - most likely this will invalidate previous tokens.

### [STEP 1] Facebook asks for `email,public_profile` permissions

- user presses `Cancel` button

  neither app nor business integration is added.

  - OmniAuth: raises error, doesn't invoke callback
  - Ueberauth: invokes callback and passes struct with error details

- user presses `Continue` button

  app is added with `email,public_profile` permissions.

  - OmniAuth: returns new access token with `email,public_profile` scopes
  - Ueberauth: returns new access token with `email,public_profile` scopes

### [STEP 2] Facebook asks for `ads_management,ads_read` permissions

these are extended permissions - that's why a separate prompt is required.
this prompt is shown iff user has granted `email,public_profile` permissions.

- user presses `Cancel` button

  app remains unchanged, business integration is not added.

  - OmniAuth: returns new access token with `email,public_profile` scopes
  - Ueberauth: returns new access token with `email,public_profile` scopes

- user presses `Continue` button

  app is removed, business integration is added with all permissions
  (`email,public_profile,ads_management,ads_read`).

  - OmniAuth: returns new access token with all scopes
  - Ueberauth: returns new access token with all scopes

troubleshooting
---------------

### callback is called twice

1. <https://stackoverflow.com/questions/37119159>
2. <https://github.com/mkdynamic/omniauth-facebook/issues/73>

callback is called twice after user either grants or declines extended
permissions - this doesn't happen when dealing with basic permissions.

- Rails

  ```
  Started GET "/auth/facebook/callback?code=AQAkrZOPRkXJ..." for 127.0.0.1 at 2018-05-11 15:07:41 +0300
  I, [2018-05-11T15:07:41.146820 #57974]  INFO -- omniauth: (facebook) Callback phase initiated.
  Processing by AuthController#callback as HTML
    Parameters: {"code"=>"AQAkrZOPRkXJ...", "state"=>"dfa379a6c9fad3b74429f5fbfaf17087ad808306d655d899", "provider"=>"facebook"}
  ...
  Completed 200 OK in 2ms (Views: 1.8ms | ActiveRecord: 0.0ms)

  Started GET "/auth/facebook/callback?code=AQAkrZOPRkXJ..." for 127.0.0.1 at 2018-05-11 15:07:41 +0300
  I, [2018-05-11T15:07:41.834104 #57974]  INFO -- omniauth: (facebook) Callback phase initiated.
  E, [2018-05-11T15:07:41.834788 #57974] ERROR -- omniauth: (facebook) Authentication failure! csrf_detected: OmniAuth::Strategies::OAuth2::CallbackError, csrf_detected | CSRF detected
  E, [2018-05-11T15:07:41.836356 #57974] ERROR -- omniauth: (facebook) Authentication failure! invalid_credentials: OmniAuth::Strategies::OAuth2::CallbackError, csrf_detected | CSRF detected
  ```

  in Rails this results into CSRF error.

- Phoenix

  ```
  14:24:26.995 [info] GET /auth/facebook/callback
  14:24:26.996 [debug] Processing with SithexWeb.AuthController.callback/2
    Parameters: %{"code" => "AQCK2OKEvra6", "provider" => "facebook"}
    Pipelines: [:browser]
  ...
  14:24:27.207 [info] Sent 200 in 211ms

  14:24:27.236 [info] GET /auth/facebook/callback
  14:24:27.236 [debug] Processing with SithexWeb.AuthController.callback/2
    Parameters: %{"code" => "AQCK2OKEvra6", "provider" => "facebook"}
    Pipelines: [:browser]
  14:24:27.456 [info] Sent 500 in 220ms
  14:24:27.459 [error] #PID<0.752.0> running SithexWeb.Endpoint terminated
  Server: sith.local:4000 (http)
  Request: GET /auth/facebook/callback?code=AQCK2OKEvra6
  ** (exit) an exception was raised:
      ** (OAuth2.Error) Server responded with status: 400

  Headers:

  ...
  www-authenticate: OAuth "Facebook Platform" "invalid_code" "This authorization code has been used."
  ...

  Body:

  %{"error" => %{"code" => 100, "fbtrace_id" => "CU56/sGwH4t", "message" => "This authorization code has been used.", "type" => "OAuthException"}}

          (oauth2) lib/oauth2/client.ex:250: OAuth2.Client.get_token!/4
          (ueberauth_facebook) lib/ueberauth/strategy/facebook.ex:48: Ueberauth.Strategy.Facebook.handle_callback!/1
          ...
  ```

  in Phoenix app this results into `This authorization code has been used.`
  because the same code is used twice to get new access token.

**solution**

TODO: not found yet.
