---
layout: post
title: Phoenix - Troubleshooting
date: 2017-08-01 17:48:52 +0300
access: public
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>


## (FunctionClauseError) no function clause matching in Plug.Conn.put_resp_header/3

<https://github.com/ueberauth/guardian/issues/196>

response header value must be a string - convert `exp` variable to string:

```elixir
conn
|> put_resp_header("authorization", "Bearer #{jwt}")
|> put_resp_header("x-expires", "#{exp}")
# ...
```

### Slogan: Kernel pid terminated (application_controller)

_/home/billing/production/billing/erl_crash.dump_ on production machine:

```
Slogan: Kernel pid terminated (application_controller) ({application_start_failure,billing,{{shutdown,{failed_to_start_child,'Elixir.BillingWeb.Endpoint',{#{'__exception__' => true,'__struct__' => 'Elixir.Run
```

**solution**

in most cases it means that `PORT` environment variable is not set.
