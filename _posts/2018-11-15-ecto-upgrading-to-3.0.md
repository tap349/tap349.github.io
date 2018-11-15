---
layout: post
title: Ecto - Upgrading to 3.0
date: 2018-11-15 18:09:12 +0300
access: public
comments: true
categories: [elixir, ecto]
---

<!-- more -->

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

if `pool_size` is set to 1, `Ecto.MigrationError` will be raised
when trying to run pending migrations:

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

NOTE: setting `pool_size` in `MyApp.Repo` module has no effect.

- set `pool_size` for all environments explicitly

  `pool_size` is 10 by default (if not set for some environment at all):

  ```elixir
  # https://github.com/elixir-ecto/ecto/blob/master/lib/ecto/repo/supervisor.ex

  @defaults [timeout: 15000, pool_timeout: 5000, pool_size: 10]
  ```

  still IMO it's better to set it expilictly just in case:

  ```elixir
  # config/dev.secret.exs
  # config/test.secret.exs

  config :lain, Lain.Repo,
    # ...
    pool_size: 10
  ```

  ```elixir
  # config/prod.secret.exs

  config :lain, Lain.Repo,
    # ...
    pool_size: 15
  ```

  or else set common `pool_size` in _config/config.exs_ and customize it
  in _config/prod.secret.exs_ only.

- bump `pool_size` in `MyApp.ReleaseTasks` module

  1. <https://hexdocs.pm/distillery/guides/running_migrations.html>

  this module is used to run migrations during deployment.

  ```diff
    # lib/lain/release_tasks.ex

    defp start_services do
      # ...
  -   Enum.each(@repos, & &1.start_link(pool_size: 1))
  +   Enum.each(@repos, & &1.start_link(pool_size: 2))
    end
  ```

remove loggers from Repo config
-------------------------------

1. <https://github.com/elixir-ecto/ecto/issues/2793>

`Ecto.Repo` has no `loggers` option any more.

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

to enable query logging in AppSignal back again:

> <https://docs.appsignal.com/elixir/integrations/phoenix.html#queries>
>
> If you're using Ecto 3, attach Appsignal.Ecto to Telemetry query events
> in your application's start/2 function.

```diff
  # lib/lain/application.ex

  def start(_type, _args) do
    import Supervisor.Spec

+   Telemetry.attach(
+     "appsignal-ecto",
+     [:my_app, :repo, :query],
+     Appsignal.Ecto,
+     :handle_event,
+     nil
+   )

    # start supervision tree
  end
```
