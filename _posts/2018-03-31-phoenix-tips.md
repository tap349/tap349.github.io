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

_config/config.exs_:

```diff
  config :billing, BillingWeb.Endpoint,
    # bind application to local IP address
+   http: [ip: {192, 168, 0, 36}, port: 4000],
```
