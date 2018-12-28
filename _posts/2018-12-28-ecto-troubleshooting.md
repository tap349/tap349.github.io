---
layout: post
title: Ecto - Troubleshooting
date: 2018-12-28 22:00:39 +0300
access: public
comments: true
categories: [elixir, ecto]
---

<!-- more -->

* TOC
{:toc}
<hr>

(DBConnection.ConnectionError) connection not available and request was dropped from queue after 171ms
------------------------------------------------------------------------------------------------------

error occurs when importing lots of entries in batches (batch size is 1000)
with `Ecto.Repo.insert_all/3`:

```
[error] Postgrex.Protocol (#PID<0.463.0>) disconnected: ** (DBConnection.ConnectionError) client #PID<0.553.0> exited
...
** (EXIT) an exception was raised:
        ** (DBConnection.ConnectionError) connection not available and request was dropped from queue after 171ms.
          You can configure how long requests wait in the queue using :queue_target and :queue_interval.
          See DBConnection.start_link/2 for more information
            (ecto_sql) lib/ecto/adapters/sql.ex:604: Ecto.Adapters.SQL.raise_sql_call_error/1
            (ecto_sql) lib/ecto/adapters/sql.ex:513: Ecto.Adapters.SQL.insert_all/8
            (ecto) lib/ecto/repo/schema.ex:56: Ecto.Repo.Schema.do_insert_all/6
            (reika) lib/reika/shopee/shop/operations/sync.ex:57: Reika.Shopee.Shop.Workers.Sync.import_shops/1
```

**solution**

1. <https://github.com/elixir-ecto/ecto/issues/2833>

> <https://github.com/elixir-ecto/ecto/issues/2833#issuecomment-440400022>
>
> pool_timeout has no effect anymore in favor of a much improved queue system.

> <https://hexdocs.pm/db_connection/DBConnection.html#start_link/2-queue-config>
>
> :queue_target in milliseconds, defaults to 50
> :queue_interval in milliseconds, defaults to 1000
>
> Our goal is to stay under :queue_target for :queue_interval. In case we
> can’t reach that, then we double the :queue_target. If we go above that,
> then we start dropping messages.

```diff
  # lib/reika/repo.ex

  defmodule Reika.Repo do
    use Ecto.Repo,
      otp_app: :reika,
-     adapter: Ecto.Adapters.Postgres
+     adapter: Ecto.Adapters.Postgres,
+     # 50ms by default
+     queue_target: 500,
+     # 1_000ms by default
+     queue_interval: 10_000
  end
```
