---
layout: post
title: Phoenix - getting started
date: 2017-07-27 13:44:01 +0300
access: private
categories: [phoenix]
---

<!-- more -->

<https://hexdocs.pm/phoenix/1.3.0-rc.3/overview.html>

install or update Hex package manager:

```sh
$ mix local.hex
```

install latest Phoenix version:

```sh
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
```

create new project without Brunch:

```sh
$ mix phx.new billing --no-brunch
```

create database users:

```sh
$ psql -d postgres
=# CREATE USER billing_dev WITH PASSWORD 'billing_dev';
=# ALTER USER billing_dev CREATEDB;
=# CREATE USER billing_test WITH PASSWORD 'billing_test';
=# ALTER USER billing_test CREATEDB;
```

edit _config/dev.exs_ and _config/test.exs_ to have correct configuration
as specified above. say, for _config/dev.exs_:

```elixir
config :billing, Billing.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "billing_dev",
  password: "billing_dev",
  database: "billing_dev",
  hostname: "localhost",
  pool_size: 10
```

create database:

```sh
$ mix ecto.create
The database for Billing.Repo has been created
```

start Phoenix app:

```sh
$ mix phx.server
```

or inside IEx:

```sh
$ iex -S mix phx.server
```

add support for Slime template engine:

- <https://github.com/slime-lang/phoenix_slime>
- <https://github.com/slime-lang/slime>
