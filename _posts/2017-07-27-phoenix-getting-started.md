---
layout: post
title: Phoenix - Getting Started
date: 2017-07-27 13:44:01 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

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
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
```

create new project
------------------

1. <https://hexdocs.pm/phoenix/up_and_running.html>

create new project without Brunch:

```sh
$ cd ~/dev
$ mix phx.new billing --no-brunch
```

setup Git repo
---------------

1. <https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/>

```sh
$ cd billing
$ git init
$ git add -A .
$ git commit -m 'initial commit'
$ get remote add origin <remote_repo_url>
$ git push
```

setup database
---------------

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

_mix.exs_:

```diff
  defp deps do
    [
      # ...
+     {:httpoison, "~> 0.12"},
+     {:distillery, "~> 1.5", runtime: false},
+     {:edeliver, "~> 1.4.4"},
+     {:timex, "~> 3.1"},
+     {:appsignal, "~> 1.4"},
+     {:jason, "~> 1.0"},
+     {:exconstructor, "~> 1.1.0"},
+     {:credo, "~> 0.9", only: [:dev, :test], runtime: false},
+     {:phoenix_expug, "~> 0.1.1"}
    ]
  end
```

### generate schemas

generate schemas (add all required columns in generated migrations -
IDK how to pass `precision` and `scale` options for decimal column to
generator so don't specify any columns altogether to be consistent):

```sh
$ mix phx.gen.schema User users
$ mix phx.gen.schema Card cards
$ mix phx.gen.schema Transfer transfers
$ mix ecto.migrate && MIX_ENV=test mix ecto.migrate
```

### generate schemas with context and supplementary modules

1. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Html.html>
2. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Json.html>
3. <https://github.com/phoenixframework/phoenix/tree/master/priv/templates>

using `phx.gen.html` or `phx.gen.json` tasks allows to generate not only
schemas but some supplementary modules as well (including context module
and `ChangesetView`).

see templates of corresponding `gen` tasks in the source code to get the
list of modules that should be generated.
