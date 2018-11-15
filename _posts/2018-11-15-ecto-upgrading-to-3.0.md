---
layout: post
title: Ecto - Upgrading to 3.0
date: 2018-11-15 18:09:12 +0300
access: private
comments: true
categories: [elixir, ecto]
---

<!-- more -->

* TOC
{:toc}
<hr>

### set adapter and pool_size in MyApp.Repo module

retrieving adapter from config files is deprecated.

```elixir
# lib/my_app/repo.ex

defmodule MyApp.Repo do
  use Ecto.Repo,
    otp_app: :my_app,
    adapter: Ecto.Adapters.Postgres,
    pool_size: 10
```

it's necessary to set `pool_size` in this module since `Ecto.Migrator`
doesn't fetch it from config files (for unknown reasons) and considers
it to be equal 1 - as a result running pending migrations when deploying
release fails with this error:

```
  stderr: ▸  Evaluation failed with: Migrations failed to run because the connection pool size is less than 2.
          ▸  Ecto requires a pool size of at least 2 to support concurrent migrators.
          ▸  When migrations run, Ecto uses one connection to maintain a lock and
          ▸  another to run migrations.
          ▸  If you are running migrations with Mix, you can increase the number
          ▸  of connections via the pool size option:
          ▸      mix ecto.migrate --pool-size 2
          ▸  If you are running the Ecto.Migrator programmatically, you can configure
          ▸  the pool size via your application config:
          ▸      config :my_app, Repo,
          ▸        ...,
          ▸        pool_size: 2 # at least
```
