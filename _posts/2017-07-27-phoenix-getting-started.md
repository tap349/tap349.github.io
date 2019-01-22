---
layout: post
title: Phoenix - Getting Started
date: 2017-07-27 13:44:01 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

install Phoenix
---------------

1. <https://hexdocs.pm/phoenix/overview.html>
2. <https://hexdocs.pm/phoenix/installation.html>

install or update Hex package manager:

```sh
$ mix local.hex
```

install latest Phoenix version:

```sh
$ mix archive.install hex phx_new 1.4.0
```

create new project
------------------

1. <https://hexdocs.pm/phoenix/up_and_running.html>

```sh
$ mix phx.new billing
```

for API-only projects:

```sh
$ mix phx.new billing --no-html --no-webpack
```

setup Git repo
---------------

1. <https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/>

```sh
$ cd billing
$ git init
$ git add -A .
$ git commit -m 'initial commit'
$ get remote add origin <REMOTE_REPO_URL>
$ git push
```

setup database
---------------

### with Docker

```yaml
# docker-compose.yml

version: '2'
services:
  db:
    image: postgres:11.1
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - 5434:5432
```

> <https://hub.docker.com/_/postgres?tab=description>
>
> POSTGRES_USER
>
> This variable will create the specified user with superuser power and
> a database with the same name.

it's possible not to specify `POSTGRES_USER` and `POSTGRES_PASSWORD` variables
in _docker-compose.yml_ at all because they default to `postgres` but IMO it's
still better to make it explicit.

```elixir
# config/dev.secret.exs

config :eva, Eva.Repo,
  username: "postgres",
  password: "postgres",
  database: "eva_dev",
  hostname: "localhost",
  port: 5434,
  pool_size: 10
```

```elixir
# config/test.secret.exs

config :eva, Eva.Repo,
  username: "postgres",
  password: "postgres",
  database: "eva_test",
  hostname: "localhost",
  port: 5434,
  pool: Ecto.Adapters.SQL.Sandbox
```

```sh
$ docker-compose up
$ mix ecto.create && MIX_ENV=test mix ecto.create
```

NOTE: don't use `postgres` user in production - create a separate application
user (this is how it's done in my Ansible tasks):

```elixir
# config/prod.secret.exs

config :eva, Eva.Repo,
  username: "eva_prod",
  password: "password",
  database: "eva_prod",
  pool_size: 15
```

### without Docker

1. <https://stackoverflow.com/a/15309551/3632318>

macOS PostgreSQL installation doesn't create `postgres` user by default
(even though `postgres` database is created):

```
$ psql -d postgres
...
postgres=# \du
                                  List of roles
Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
tap       | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

create him manually:

```
$ psql -d postgres
...
postgres=# CREATE USER postgres WITH SUPERUSER PASSWORD 'postgres';
CREATE ROLE
```

`postgres` user is used by default in generated _config/dev.exs_ and
_config/test.exs_ files so just leave these defaults.

_config/dev.exs_ (for example):

```elixir
config :billing, Billing.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "postgres",
  password: "postgres",
  database: "billing_dev",
  pool_size: 10
```

```sh
$ mix ecto.create && MIX_ENV=test mix ecto.create
```

start app
---------

```sh
$ mix phx.server
```

or inside IEx:

```sh
$ iex -S mix phx.server
```

customize app
-------------

1. <https://hexdocs.pm/phoenix/adding_pages.html>

### add useful packages

for example:

```diff
  _mix.exs_

  defp deps do
    [
      # ...
+     {:appsignal, "~> 1.5"},
+     {:credo, "~> 0.9", only: [:dev, :test], runtime: false},
+     {:distillery, "~> 1.5", runtime: false},
+     {:edeliver, "~> 1.4"},
+     {:exconstructor, "~> 1.1.0"},
+     {:httpoison, "~> 1.0"},
+     {:jason, "~> 1.0"},
+     {:phoenix_slime, "~> 0.10.0"},
+     {:timex, "~> 3.1"}
    ]
  end
```

### generate schemas

1. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Schema.html>
2. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Html.html>
3. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Json.html>
4. <https://github.com/phoenixframework/phoenix/tree/master/priv/templates>

schemas only:

```sh
$ mix phx.gen.schema User users
$ mix phx.gen.schema Card cards
$ mix phx.gen.schema Transfer transfers
$ mix ecto.migrate
```

schemas with context and supplementary modules (including `ChangesetView`):

```sh
$ mix phx.gen.json Page Conversation conversations external_id:string \
  labels:array:string created_time:utc_datetime updated_time:utc_datetime
$ mix ecto.migrate
```

it's not necessary to run `MIX_ENV=test mix ecto.migrate` since this alias
is added by default in a newly generated Phoenix project:

```elixir
# mix.exs

defmodule Lain.Mixfile do
  # ...
  defp aliases do
    [
      # ...
      test: ["ecto.create --quiet", "ecto.migrate", "test"]
    ]
  end
end
```
