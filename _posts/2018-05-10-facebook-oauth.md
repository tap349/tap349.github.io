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

request information about user in browser
-----------------------------------------

the request is made on behalf of your app (OAuth implementation would add
`client_id` query param to this URL):

```
http://sith.local:4000/auth/facebook?scope=email,public_profile
```

the 1st time you make a request Facebook will ask currently signed in user
for specified permissions (to view email address in this case):

> \<my_app> will receive: your email address

on the 2nd and subsequent requests Facebook will remember user choice and
will return requested information without prompting user.

NOTE: a new token will be returned on each request which will invalidate
      a previous one (I guess).

TODO: IDK yet how to revoke access to information about user (so that user
      is prompted again).
