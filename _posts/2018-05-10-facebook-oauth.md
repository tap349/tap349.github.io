---
layout: post
title: Facebook - OAuth
date: 2018-05-10 13:06:44 +0300
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

  <dt>request URL</dt>
  <dd>http://my-app.com:4000/auth/facebook</dd>

  <dt>authorize URL (aka authorization URL)</dt>
  <dd>https://www.facebook.com/dialog/oauth</dd>

  <dt>redirect URI (aka callback URL)</dt>
  <dd>http://my-app.com:4000/auth/facebook/callback</dd>
</dl>

<hr>

1. <https://paw.cloud/docs/examples/facebook-api>

## whitelist redirect URI

1. <https://help.sharetribe.com/managing-your-marketplace/social-media/how-to-solve-the-url-blocked-this-redirect-failed-because-facebook-login-error>
2. <https://stackoverflow.com/questions/2459728/how-to-test-facebook-connect-locally>

it's necessary to whitelist redirect URI - all other redirect URIs will be
blocked by Facebook:

> URL Blocked: This redirect failed because the redirect URI is not whitelisted
> in the app’s Client OAuth Settings. Make sure Client and Web OAuth Login are
> on and add all your app domains as Valid OAuth Redirect URIs.

### add local domain to /etc/hosts

```diff
  # /etc/hosts

+ 127.0.0.1 sith.local
```

or else it's possible to whitelist `localhost` as a valid redirect URI in FD
without using any custom local domains.

> <https://stackoverflow.com/a/5626979/3632318>
>
> Facebook does not "connect" back to your server. Their JS does. And the JS
> runs in the context of your browser. Which knows where "localhost" points to.

### whitelist local domain in FD

> <https://wp-native-articles.com/blog/news/how-to-fix-facebook-apps-error-cant-load-url-domain-url-isnt-included-apps-domains/>
>
> Any new Facebook Login Apps create AFTER the beginning of March 2018 now have
> Use Strict Mode for Redirect URIs and Enforce HTTPS enabled by default and can
> no longer be disabled.
>
> ...it means that you now have to put the exact return URL into the Valid OAuth
> Redirect URIs input. Previously, with strict mode disabled, you could just put
> your domain name in and that would be enough.

=> you must use exact redirect URI - you can't just enter domain name as a valid
redirect URI (say, `http://sith.local`) or else you'll get this error:

> Can't Load URL: The domain of this URL isn't included in the app's domains. To
> be able to load this URL, add all domains and subdomains of your app to the
> App Domains field in your app settings.

<!-- prettier-ignore -->
| FD: `PRODUCTS` (section in left menu) → `Facebook Login` → `Settings`
| `Client OAuth Settings` (section)

- `Valid OAuth Redirect URIs` (input): add
  `http://sith.local:4000/auth/facebook/callback`

## request user for permissions

1. <https://developers.facebook.com/docs/marketing-api/access#manually-getting-access-tokens>

NOTE: scopes = permissions.

### how OAuth library works

both OmniAuth and Ueberauth work alike under the hood:

- user clicks request URL

  ```
  http://my-app.com:4000/auth/facebook
  ```

  this link might already contain requested scopes:

  ```
  http://example.com:4000/auth/facebook?scope=email,public_profile,ads_management,ads_read
  ```

  otherwise OAuth library will add default scopes on the next step.

- OAuth library redirects to authorize URL

  OAuth library adds query params to authorize URL including `client_id` so that
  the request is made on behalf of your app:

  ```
  https://www.facebook.com/dialog/oauth?
    client_id=<YOUR_APP_ID>
    &redirect_uri=<YOUR_REDIRECT_URL>
    &scope=ads_management
  ```

- Facebook redirects to callback URL

  ```
  http://YOUR_REDIRECT_URL?code=<AUTHORIZATION_CODE>
  ```

  this happens when user finishes authentication flow by granting or declining
  requested permissions. in the former case Facebook adds `code` query param
  (authorization code).

- OAuth library makes request to access token URL

  OAuth library intercepts Facebook request with authorization code in query
  params and uses it to fetch new access token:

  ```
  https://graph.facebook.com/oauth/access_token?
    client_id=<YOUR_APP_ID>
    &redirect_uri=<YOUR_REDIRECT_URL>
    &client_secret=<YOUR_APP_SECRET>
    &code=<AUTHORIZATION_CODE>
  ```

  if Facebook request doesn't have `code` query param, OAuth library can either
  raise error (OmniAuth) or populate error data structure (Ueberauth).

- OAuth library populates auth data structure

  when OAuth library fetches access token, it continues processing Facebook
  request to callback URL and populates auth data structure (`%Ueberauth.Auth`
  in Ueberauth, `request.env['omniauth.auth']` in OmniAuth) with access token
  and other requested information about user which can be used later to find or
  create user and sign him in.

### server-side authentication flow in Facebook

1. <https://developers.gigya.com/display/GD/Facebook+Login+Permissions#FacebookLoginPermissions-AvailablePermissions>

there are 2 possible outcomes when user is prompted by Facebook to give
permissions - either app or business integration can be added to user's Facebook
account:

- app with data access

  <!-- prettier-ignore -->
  | Facebook: `▼` (top right menu) → `Settings`
  | `Apps and Websites` (left menu) → `Active` (tab)

  added app will have `email,public_profile` scopes.

- business integration with data and account access

  <!-- prettier-ignore -->
  | Facebook: `▼` (top right menu) → `Settings`
  | `Business Integrations` (left menu) → `Active` (tab)

  added business integration will have all permissions
  (`email,public_profile,ads_management,ads_read`).

corresponding permissions might be revoked by removing either app or business
integration from user's Facebook account.

any successful response from Facebook contains, inter alia, persistent access
token (JWT token) but its scopes may be different based on user action (`scopes`
is a JWT token field). if access token with requested scopes has been created
before, Facebook won't prompt user next time request is made but will return a
new token with the same scopes - this will invalidate previous tokens most
likely.

- [STEP 1] Facebook asks for `email,public_profile` permissions

  - user presses `Cancel` button

    neither app nor business integration is added.

    - OmniAuth: raises error, doesn't invoke callback
    - Ueberauth: invokes callback and passes struct with error details

  - user presses `Continue` button

    app is added with `email,public_profile` permissions.

    - OmniAuth: returns new access token with `email,public_profile` scopes
    - Ueberauth: returns new access token with `email,public_profile` scopes

- [STEP 2] Facebook asks for `ads_management,ads_read` permissions

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
