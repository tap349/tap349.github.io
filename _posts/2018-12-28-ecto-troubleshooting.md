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
```

**solution**

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
  # lib/my_app/repo.ex

  defmodule MyApp.Repo do
    use Ecto.Repo,
      otp_app: :my_app,
-     adapter: Ecto.Adapters.Postgres
+     adapter: Ecto.Adapters.Postgres,
+     # 50ms by default
+     queue_target: 5_000,
+     # 1_000ms by default
+     queue_interval: 100_000
  end
```

(DBConnection.ConnectionError) client timed out because it queued and checked out the connection for longer than 15000ms
------------------------------------------------------------------------------------------------------------------------

```
iex> MyApp.User.Mutator.delete_all()
[error] Postgrex.Protocol (#PID<0.439.0>) disconnected: ** (DBConnection.ConnectionError)
  client #PID<0.478.0> timed out because it queued and checked out the connection for longer than 15000ms
** (DBConnection.ConnectionError) tcp recv: closed (the connection was closed by the pool,
  possibly due to a timeout or because the pool has been terminated)
    (ecto_sql) lib/ecto/adapters/sql.ex:604: Ecto.Adapters.SQL.raise_sql_call_error/1
    (ecto_sql) lib/ecto/adapters/sql.ex:537: Ecto.Adapters.SQL.execute/5
```

**solution**

increase the value of `timeout` option (time to wait for query call to finish):

> <https://github.com/elixir-ecto/ecto/issues/1838#issuecomment-265692336>
>
> You can either Ecto.Adapters.SQL.query(Repo, query, params, timeout: 30_000)
> or by setting the timeout: 30_000 in your Repo configuration in your config/*
> files.

- on per a repository operation basis

  > <https://hexdocs.pm/ecto/Ecto.Repo.html#module-shared-options>
  >
  > Almost all of the repository operations below accept the following options:
  >
  > :timeout - The time in milliseconds to wait for the query call to finish,
  >   :infinity will wait indefinitely (default: 15000);

  ```elixir
  MyApp.Repo.delete_all(MyApp.User, timeout: 30_000)
  ```

- for all repository operations via repository configuration

  > <https://hexdocs.pm/ecto/2.0.0/Ecto.Adapters.Postgres.html>
  >
  > Compile time options
  >
  > :timeout - The default timeout to use on queries, defaults to 15000

  ```elixir
  # config/prod.secret.exs

  config :my_app, MyApp.Repo,
    username: "my_app_prod",
    password: "my_app_prod",
    database: "my_app_prod",
    pool_size: 30,
    timeout: 30_000
  ```

  NOTE: setting `timeout` option in `MyApp.Repo` module has no effect:

  ```elixir
  # lib/my_app/repo.ex

  defmodule MyApp.Repo do
    use Ecto.Repo,
      otp_app: :my_app,
      adapter: Ecto.Adapters.Postgres,
      timeout: 120_000
  end
  ```

set breakpoint in `Ecto.Adapters.SQL.execute/5` to see actual options being
used when performing repository operation:

```elixir
# deps/ecto_sql/lib/ecto/adapters/sql.ex

def execute(adapter_meta, query_meta, prepared, params, opts) do
  require IEx; IEx.pry
  # print `adapter_meta`
  %{num_rows: num, rows: rows} =
    execute!(adapter_meta, prepared, params, put_source(opts, query_meta))

  {num, rows}
end
```
