---
layout: post
title: Phoenix - Authentication
date: 2017-08-01 10:50:39 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

API
---

### manual implementation

1. <https://m.alphasights.com/simple-web-servers-with-plug-and-cowboy-34f7a174f252>
2. <http://learningwithjb.com/posts/authenticating-users-using-a-token-with-phoenix>

### using `Phoenix.Token`

1. <https://hexdocs.pm/phoenix/Phoenix.Token.html>
2. <https://elixirforum.com/t/how-is-phoenix-token-different-from-jwt/2349/4>
3. <https://dennisreimann.de/articles/phoenix-passwordless-authentication-magic-link.html>

### using Guardian and JWT

1. <https://github.com/ueberauth/guardian>
2. <http://blog.overstuffedgorilla.com/simple-guardian-api-authentication/>

#### signing out

1. <http://blog.overstuffedgorilla.com/simple-guardian-api-authentication/#logout>

<https://elixirforum.com/t/guardian-jwt-vs-phoenix-token/853/29>:

> The first question is whether you need to revoke the jwt at all -
> in many cases, it might be enough to just let it expire. With many
> apis, there isn’t really a logout functionality - the user will just
> access the resources he / she needs and then stop using it.
> If you do want to revoke jwts, I know that many developers use Redis for
> this, and that might be quicker than a db lookup.

that is you need external storage to revoke JWT token before it expires -
say, `Guardian DB` package allows to track tokens in database when using
`Guardian`.

<https://github.com/ueberauth/guardian_db#disadvantages>:

> With Guardian.DB, every request requires a trip to the database,
> as Guardian now needs to ensure that a record of the token exists.
> This can arguably eliminate the main advantage of using a JWT
> authentication solution, which is statelessness.

when using JWT token for browser sessions, just flush the session.

basic authentication
--------------------

1. <http://radekmolenda.github.io/2016/02/26/phoenix-basic-auth-exercise.html>
2. <https://medium.com/@paulfedory/basic-authentication-in-your-phoenix-app-fa24e57baa8>
