---
layout: post
title: Facebook - OAuth
date: 2018-05-10 13:06:44 +0300
access: public
comments: true
categories: [facebook]
---

<!-- more -->

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

  OAuth library adds query params to authorize URL including
  `client_id` so that the request is made on behalf of your app:

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

  this happens when user finishes authentication flow by granting or
  declining requested permissions. in the former case Facebook adds
  `code` query param (authorization code).

- OAuth library makes request to access token URL

  OAuth library intercepts Facebook request with authorization code
  in query params and uses it to fetch new access token:

  ```
  https://graph.facebook.com/oauth/access_token?
    client_id=<YOUR_APP_ID>
    &redirect_uri=<YOUR_REDIRECT_URL>
    &client_secret=<YOUR_APP_SECRET>
    &code=<AUTHORIZATION_CODE>
  ```

  if Facebook request doesn't have `code` query param, OAuth library
  can either raise error (OmniAuth) or populate error data structure
  (Ueberauth).

- OAuth library populates auth data structure

  when OAuth library fetches access token, it continues processing
  Facebook request to callback URL and populates auth data structure
  (`%Ueberauth.Auth` in Ueberauth, `request.env['omniauth.auth']` in
  OmniAuth) with access token and other requested information about
  user which can be used later to find or create user and sign him in.

### server-side authentication flow in Facebook

1. <https://developers.gigya.com/display/GD/Facebook+Login+Permissions#FacebookLoginPermissions-AvailablePermissions>

there are 2 possible outcomes when user is prompted by Facebook to give
permissions - either app or business integration can be added to user's
Facebook account:

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

any successful response from Facebook contains, inter alia, persistent
access token (JWT token) but its scopes may be different based on user
action (`scopes` is a JWT token field). if access token with requested
scopes has been created before, Facebook won't prompt user next time
request is made but will return a new token with the same scopes - this
will invalidate previous tokens most likely.

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

troubleshooting
---------------

### callback is called twice

1. <https://stackoverflow.com/questions/37119159>
2. <https://github.com/mkdynamic/omniauth-facebook/issues/73>

callback is called twice after user either grants or declines extended
permissions - this doesn't happen when dealing with basic permissions.

this error doesn't occur if user has already granted permissions before
and Facebook just returns a new access token with the same scopes.

- Rails

  ```
  Started GET "/auth/facebook/callback?code=<code>&state=<state>" for 127.0.0.1 at 2018-05-11 15:07:41 +0300
  INFO -- omniauth: (facebook) Callback phase initiated.
  Processing by AuthController#callback as HTML
    Parameters: {"code"=>"<code>", "state"=>"dfa379a6c9fad3b74429f5fbfaf17087ad808306d655d899", "provider"=>"facebook"}
  ...
  Completed 200 OK in 2ms (Views: 1.8ms | ActiveRecord: 0.0ms)

  Started GET "/auth/facebook/callback?code=<code>" for 127.0.0.1 at 2018-05-11 15:07:41 +0300
  INFO -- omniauth: (facebook) Callback phase initiated.
  ERROR -- omniauth: (facebook) Authentication failure! csrf_detected: OmniAuth::Strategies::OAuth2::CallbackError, csrf_detected | CSRF detected
  ERROR -- omniauth: (facebook) Authentication failure! invalid_credentials: OmniAuth::Strategies::OAuth2::CallbackError, csrf_detected | CSRF detected
  ```

  in Rails this results into CSRF error.

- Phoenix

  {% raw %}
  ```
  [info] GET /auth/facebook/callback
  [debug] Processing with SithexWeb.AuthController.callback/2
    Parameters: %{"code" => "AQCK2OKEvra6", "provider" => "facebook"}
    Pipelines: [:browser]
  ...
  [info] Sent 200 in 211ms

  [info] GET /auth/facebook/callback
  [debug] Processing with SithexWeb.AuthController.callback/2
    Parameters: %{"code" => "AQCK2OKEvra6", "provider" => "facebook"}
    Pipelines: [:browser]
  [info] Sent 500 in 220ms
  [error] #PID<0.752.0> running SithexWeb.Endpoint terminated
  Server: sith.local:4000 (http)
  Request: GET /auth/facebook/callback?code=<code>
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
  {% endraw %}

  in Phoenix app this results into `This authorization code has been used.`
  because the same code is used twice to get new access token.

**solution**

most likely errors in Rails and Phoenix are caused by Facebook calling
callback URL twice but are still different.

- OmniAuth (Rails)

  1. <https://github.com/ShippingEasy/omniauth-ecwid/issues/2>
  2. <https://github.com/omniauth/omniauth-oauth2/issues/32>

  OmniAuth uses `state` authentication parameter to help mitigate CSRF attacks
  (see [The State Parameter](https://auth0.com/docs/protocols/oauth2/oauth-state)).

  before making request to authorize URL OmniAuth creates `omniauth.state`
  session variable whose value is a randomnly generated string.

  OmniAuth adds `state` query param with this value to authorize URL,
  this param is then returned by Facebook in his request to callback URL:

  ```
  Started GET "/auth/facebook/callback?code=<code>&state=<state>"
  ```

  in callback phase OmniAuth tries to verify the value of `state` query param
  against the value stored in session - if they match, OmniAuth makes a request
  to obtain new access token. when comparing `state` values, OmniAuth removes
  `omniauth.state` session variable from session storage so that another request
  to callback URL with the same `state` cannot be made.

  CSRF attack will be detected if `state` values don't match - this is what
  happens when duplicate request from Facebook is received: `omniauth.auth`
  session variable has been already removed from session storage by the 1st
  request => `state` query param value doesn't match `nil`:

  ```ruby
  # https://github.com/omniauth/omniauth-oauth2/blob/ee63077b1c3f677f0042010e393e9fd0bf1d69d2/lib/omniauth/strategies/oauth2.rb#L66

  def callback_phase
    error = request.params["error_reason"] || request.params["error"]
    if error
      # ...
    elsif !options.provider_ignores_state && (request.params["state"].to_s.empty? || request.params["state"] != session.delete("omniauth.state"))
      fail!(:csrf_detected, CallbackError.new(:csrf_detected, "CSRF detected"))
    else
      self.access_token = build_access_token
      self.access_token = access_token.refresh! if access_token.expired?
      super
    end
    # ...
  end
  ```

  workaround (as seen from the code) is to disable checking `state` at all
  with `provider_ignores_state` option:

  ```ruby
  # config/initializers/omniauth.rb

  Rails.application.config.middleware.use OmniAuth::Builder do
    provider :facebook, app_id, app_secret,
      # ...
      provider_ignores_state: true
  end
  ```

  even though it's not recommended since it makes your application susceptible
  to CSRF attacks but I haven't found better solution so far.

  in previous versions of OmniAuth library (prior to 4.0.0, current version
  is 5.0.0) this problem had different origin so solutions proposed in these
  issues no longer work:

  - <https://github.com/mkdynamic/omniauth-facebook/issues/276>
  - <https://github.com/mkdynamic/omniauth-facebook/issues/278>
  - <https://github.com/mkdynamic/omniauth-facebook/issues/284>

  ***UPDATE***

  BTW after checking `state` is disabled, we get the same error as in Phoenix
  app:

  ```
  Started GET "/auth/facebook/callback?code=<code>&state=<state>" for 127.0.0.1 at 2018-05-21 17:53:46 +0300
  INFO -- omniauth: (facebook) Callback phase initiated.
  ERROR -- omniauth: (facebook) Authentication failure! invalid_credentials: OAuth2::Error, {"message"=>"This authorization code has been used.", "type"=>"OAuthException", "code"=>100, "fbtrace_id"=>"AYGVd7HbMhI"}:
  {"error":{"message":"This authorization code has been used.","type":"OAuthException","code":100,"fbtrace_id":"AYGVd7HbMhI"}}

  OAuth2::Error - {"message"=>"This authorization code has been used.", "type"=>"OAuthException", "code"=>100, "fbtrace_id"=>"AYGVd7HbMhI"}:
  {"error":{"message":"This authorization code has been used.","type":"OAuthException","code":100,"fbtrace_id":"AYGVd7HbMhI"}}:
  ```

- Ueberauth library (Phoenix)

  Ueberauth doesn't use `state` authentication parameter to prevent CSRF
  attacks like OmniAuth does (that is Ueberauth doesn't add `state` query
  param to authorize URL) but when Ueberauth makes the 2nd request to get
  access token using the same code, Facebook returns `This authorization
  code has been used` error as might be expected.

eventually the problem must be with Facebook Graph API making double request
to callback URL - not with any OAuth library (OmniAuth or Ueberauth).

### The parameter app_id is required

Facebook response when making request to request URL:

> The parameter app_id is required

**solution**

this means `client_id` query param has no value or missing.

in my case I misspelled environment variable name in OmniAuth initializer:

```diff
  # config/initializers/omniauth.rb

  Rails.application.config.middleware.use OmniAuth::Builder do
-   provider :facebook, ENV['FACEBOOK_APP_KEY'], ENV['FACEBOOK_APP_SECRET'],
+   provider :facebook, ENV['FACEBOOK_APP_ID'], ENV['FACEBOOK_APP_SECRET'],
    display: 'page',
    info_fields: 'name,email,first_name,last_name',
    scope: 'email,public_profile,ads_management,ads_read'
  end
```

as a result `client_id` query param had no value:

```
...?client_id&redirect_uri=...
```

### unknown error in GAE

response to sample request (`me?fields=id,name`) in GAE when access token
obtained from another user is used:

```json
{
  "error": {
    "code": 1,
    "error_subcode": 1357045,
    "message": "unknown error (empty response)",
    "type": "http",
    "status": 0
  }
}
```

**solution**

error occurs in Chrome browser only (I guess because of numerous blocker
browser extensions installed) - try to use another browser (say, Safari).

***UPDATE***

error is caused by ad blockers - disable these extensions and reload the
page. or else it's better to whitelist GAE page in their settings.

### You are not logged in

Facebook response when making request to request URL:

> You are not logged in: You are not logged in. Please log in and try again.

**solution**

this means that passed redirect URI is not whitelisted (see above).

### [OmniAuth] OmniAuth::Strategies::Facebook::NoAuthorizationCodeError

this error occurs when user presses `Cancel` on Facebook Login screen:

```
Started GET "/auth/facebook/callback?error=access_denied&error_code=200&error_description=Permissions%20error&error_reason=user_denied&state=<state>" for 127.0.0.1 at 2018-06-01 14:29:37 +0300
INFO -- omniauth: (facebook) Callback phase initiated.
ERROR -- omniauth: (facebook) Authentication failure! no_authorization_code: OmniAuth::Strategies::Facebook::NoAuthorizationCodeError, must pass either a `code` (via URL or by an `fbsr_XXX` signed request cookie)
```

**solution**

1. <https://github.com/omniauth/omniauth/wiki/FAQ#omniauthfailureendpoint-does-not-redirect-in-development-mode>

> By default, OmniAuth 1.1.0 and later raises an exception in development mode
> when authentication fails.

=> in case of authentication failure OmniAuth raises exception in development
mode and redirets to failure page in production mode.

sample failure page URL:

```
https://my-app.com/auth/failure?message=no_authorization_code&strategy=facebook#_=_
```

configure OmniAuth `on_failure` behaviour to redirect to failure page in
development mode as well:

```ruby
# config/initializers/omniauth.rb

OmniAuth.config.on_failure = proc do |env|
  OmniAuth::FailureEndpoint.new(env).redirect_to_failure
end
```

make sure to add `/auth/failure` prior to other auth routes:

```ruby
# config/routes.rb

get '/auth/failure', to: 'auth#failure', as: 'auth_failure'
get '/auth/:provider', to: 'auth#request', as: 'auth_request'
get '/auth/:provider/callback', to: 'auth#callback', as: 'auth_callback'
```

if OmniAuth is already configured to redirect to failure page in development
mode and `/auth/failure` route is placed after other auth routes or missing
at all, you'll get some weird error about non-existing `request` controller
action:

```
Started GET "/auth/facebook/callback?error=access_denied&error_code=200&error_description=Permissions%20error&error_reason=user_denied&state=<state>" for 127.0.0.1 at 2018-06-01 15:27:37 +0300
INFO -- omniauth: (facebook) Callback phase initiated.
ERROR -- omniauth: (facebook) Authentication failure! no_authorization_code: OmniAuth::Strategies::Facebook::NoAuthorizationCodeError, must pass either a `code` (via URL or by an `fbsr_XXX` signed request cookie)
Started GET "/auth/failure?message=no_authorization_code&strategy=facebook" for 127.0.0.1 at 2018-06-01 15:27:37 +0300

AbstractController::ActionNotFound - The action 'request' could not be found for AuthController:
  appsignal (2.6.1) lib/appsignal/rack/rails_instrumentation.rb:19:in `call'
```

BTW Ueberauth doesn't raise error in both environments at all but adds
`ueberauth_failure` key with populated `%Ueberauth.Failure{}` struct to
`assigns` map - this key can be pattern matched in controller action to
detect authentication failure.
