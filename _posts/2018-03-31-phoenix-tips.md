---
layout: post
title: Phoenix - Tips
date: 2018-03-31 14:19:58 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

make application available from outside
---------------------------------------

say, it might be useful to make application available on local network
in development so that it can be tested by others.

```diff
  # config/config.exs

  config :billing, BillingWeb.Endpoint,
+   # bind application to local IP address
+   http: [ip: {192, 168, 0, 36}, port: 4000],
```

using SSL
---------

1. <https://hexdocs.pm/phoenix/endpoint.html#using-ssl>
2. <http://ohanhi.com/phoenix-ssl-localhost.html>
3. <https://gist.github.com/tadast/9932075>

the idea is that it's possible to generate private key and self-signed
certificate and specify them in `https` key of endpoint configuration.

umbrella apps vs. contexts
--------------------------

> <https://www.reddit.com/r/elixir/comments/6bpah8/phoenix_context_vs_elixir_umbrella_apps/dhprpl5/>
>
> The rule of thumb is to use umbrellas where you can use separate storage
> and contexts everywhere else. Many quite big apps use only one database.
> It doesn't make sense to configure them separately in Umbrellas and you
> can't stop one part without bringing entire system down. It doesn't make
> sense to extract them to umbrellas, but you still don't want to tangle
> stuff too much.
