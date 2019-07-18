---
layout: post
title: Ecto - Upgrading to 3.0
date: 2018-11-15 18:09:12 +0300
access: public
comments: true
categories: [elixir, ecto]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

set adapter in MyApp.Repo module
--------------------------------

retrieving adapter from config files is deprecated.

**solution**

```elixir
# lib/my_app/repo.ex

defmodule MyApp.Repo do
  use Ecto.Repo,
    otp_app: :my_app,
    adapter: Ecto.Adapters.Postgres
```

increase connection pool size to at least 2
-------------------------------------------

if `pool_size` is set to 1, `Ecto.MigrationError` will be raised when trying
to run pending migrations:

```elixir
# https://github.com/elixir-ecto/ecto_sql/blob/master/lib/ecto/adapters/sql.ex

def lock_for_migrations(meta, query, opts, fun) do
 %{opts: default_opts} = meta

  if Keyword.fetch(default_opts, :pool_size) == {:ok, 1} do
    raise_pool_size_error()
  end

  # ...
end

defp raise_pool_size_error do
  raise Ecto.MigrationError, """
  Migrations failed to run because the connection pool size is less than 2.
  Ecto requires a pool size of at least 2 to support concurrent migrators.
  When migrations run, Ecto uses one connection to maintain a lock and
  another to run migrations.
  If you are running migrations with Mix, you can increase the number
  of connections via the pool size option:
      mix ecto.migrate --pool-size 2
  If you are running the Ecto.Migrator programmatically, you can configure
  the pool size via your application config:
      config :my_app, Repo,
        ...,
        pool_size: 2 # at least
  """
end
```

**solution**

make sure `pool_size` is at least 2 in all environments.

- set `pool_size` for all environments explicitly

  setting `pool_size` in `MyApp.Repo` module has no effect - it must be
  configured in environment config files.

  `pool_size` is 10 by default (if not set for some environment at all):

  ```elixir
  # https://github.com/elixir-ecto/ecto/blob/master/lib/ecto/repo/supervisor.ex

  @defaults [timeout: 15_000, pool_timeout: 5000, pool_size: 10]
  ```

  still IMO it's better to set it expilictly just in case:

  ```elixir
  # config/dev.secret.exs
  # config/test.secret.exs
  # config/prod.secret.exs

  config :my_app, MyApp.Repo,
    # ...
    pool_size: 15
  ```

  or else set common `pool_size` in _config/config.exs_ and customize it
  in _config/prod.secret.exs_ only.

- bump `pool_size` in `MyApp.ReleaseTasks` module

  1. <https://hexdocs.pm/distillery/guides/running_migrations.html>

  this module is used to run migrations during deployment.

  ```diff
    # lib/my_app/release_tasks.ex

    defp start_services do
      # ...
  -   Enum.each(@repos, & &1.start_link(pool_size: 1))
  +   Enum.each(@repos, & &1.start_link(pool_size: 2))
    end
  ```

use telemetry events instead of loggers
---------------------------------------

> <https://github.com/elixir-ecto/ecto/blob/master/CHANGELOG.md#v300-2018-10-29>
>
> [Ecto.Repo] The :loggers configuration is deprecated in favor of "Telemetry
> Events"

```
$ iex
...
warning: the :loggers configuration for MyApp.Repo is deprecated.

  * To customize the log level, set log: :debug | :info | :warn | :error instead
  * To disable logging, set log: false instead
  * To hook into logging events, see the "Telemetry Events" section in Ecto.Repo docs

  lib/ecto/repo/supervisor.ex:76: Ecto.Repo.Supervisor.compile_config/2
  lib/my_app/repo.ex:2: (module)
  ...
```

> <https://hexdocs.pm/ecto/Ecto.Repo.html#module-telemetry-events>
>
> We recommend adapters to publish certain Telemetry events listed below.

**solution**

```diff
  # config/prod.secret.exs

  config :my_app, MyApp.Repo,
    # ...
-   loggers: [Appsignal.Ecto, Ecto.LogEntry]
```

### AppSignal

to enable query logging in AppSignal back again:

> <https://docs.appsignal.com/elixir/integrations/phoenix.html#queries>
>
> If you're using Ecto 3, attach Appsignal.Ecto to Telemetry query events
> in your application's start/2 function.

```diff
  # lib/my_app/application.ex

  def start(_type, _args) do
    import Supervisor.Spec

+   :telemetry.attach(
+     "appsignal-ecto",
+     [:my_app, :repo, :query],
+     &Appsignal.Ecto.handle_event/4,
+     nil
+   )

    # start supervision tree
  end
```

### MyApp.ReleaseTasks

1. <https://github.com/elixir-ecto/ecto/issues/2793>

make sure to start `ecto_sql` instead of `ecto` in `MyApp.ReleaseTasks` module:

```diff
  # lib/my_app/release_tasks.ex

  @start_apps [
    :crypto,
    :ssl,
    :postgrex,
-   :ecto
+   :ecto_sql
  ]

  # ...
```

or else `telemetry` application won't be started either and you'll get this
error if any `Ecto.LogEntry` is attempted to be logged when running pending
migrations:

```
[172.XXX.XXX.XX] module=DBConnection [error] an exception was raised logging
%DBConnection.LogEntry{...}: ** (ArgumentError) argument error
[172.XXX.XXX.XX]     (stdlib) :ets.lookup(Telemetry.HandlerTable, [:my_app, :repo, :query])
[172.XXX.XXX.XX]     (telemetry) lib/telemetry/handler_table.ex:59: Telemetry.HandlerTable.list_for_event/1
[172.XXX.XXX.XX]     (telemetry) lib/telemetry.ex:76: Telemetry.execute/3
[172.XXX.XXX.XX]     (ecto_sql) lib/ecto/adapters/sql.ex:756: Ecto.Adapters.SQL.log/4
[172.XXX.XXX.XX]     (db_connection) lib/db_connection.ex:1303: DBConnection.log/5
[172.XXX.XXX.XX]     (db_connection) lib/db_connection.ex:1359: DBConnection.run_transaction/4
[172.XXX.XXX.XX]     (ecto_sql) lib/ecto/migrator.ex:184: Ecto.Migrator.do_run_maybe_in_transaction/3
[172.XXX.XXX.XX]     (elixir) lib/task/supervised.ex:89: Task.Supervised.do_apply/2
```
