---
layout: post
title: Elixir - Deployment
date: 2017-08-02 21:38:18 +0300
access: public
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

complete guides:

1. <https://elixirforum.com/t/elixir-deployment-tools-general-discussion-blog-posts-wiki/827>
2. <https://hackernoon.com/state-of-the-art-in-deploying-elixir-phoenix-applications-fe72a4563cd8>
3. <https://dustinfarris.gitbooks.io/phoenix-continuous-deployment/content/>
4. <https://jimmy-beaudoin.com/posts/elixir/phoenix-deployment/>
5. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>
6. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>
7. <https://habrahabr.ru/post/320096/>

NOTE: all paths on production machine are specified relative to
      application directory located at _$DELIVER_TO/\<app\_name\>/_.

## prepare for deployment

1. <https://hexdocs.pm/phoenix/deployment.html>

### secrets

1. <https://hexdocs.pm/phoenix/deployment.html#handling-of-your-application-secrets>

- replace all values in _config/prod.exs_ with environment variables and set
  those variables on production machine OR
- hard-code secrets in _config/prod.exs_ and place it on production machine
  manually or via Chef, say, at _/var/prod.secret.exs_

  _config/prod.exs_:

  ```diff
  - import_config "config/prod.secret.exs"
  + import_config "/var/prod.secret.exs"
  ```

### assets

<https://hexdocs.pm/phoenix/deployment.html#compiling-your-application-assets>:

> If you are not serving or don’t care about assets at all, you can just remove
> the cache_static_manifest configuration from config/prod.exs.

so if application doesn't deal with assets remove this line in _config/prod.exs_:

```diff
config :billing, BillingWeb.Endpoint,
  load_from_system_env: true,
+ url: [host: "example.com", port: 80]
- url: [host: "example.com", port: 80],
- cache_static_manifest: "priv/static/cache_manifest.json"
```

### artifacts (say, YAML files)

<https://elixirforum.com/t/including-data-files-in-a-distillery-release/2813>:

> The traditional place to put non-code resources that are needed at runtime is the
> priv folder. All the tools are aware of this convention and preserve proper paths.
>
> You can access the files at runtime using Application.app_dir(app_name, "priv/path/to/file")

<https://elixirforum.com/t/is-it-possible-to-include-resource-files-when-packaging-my-project-using-mix-escript/730/4>:

> "priv" is like "OTP" where its name made sense at the beginning but today
> it has grown beyond that. All it matters now is that we put in the "priv"
> directory any artifact that you need in production alongside your code.

<https://stackoverflow.com/a/32097896>:

> Elixir applications care about two directories: 1. ebin (which is where you
> put compiled code) and 2. priv (auxiliary files that you need to run your
> software in production, like static files). If you rely on a file that is not
> in any of those directories, things can break when running in production or
> building releases.

in some Elixir module:

```elixir
defmodule Neko.Reader do
  @rules_path Application.app_dir(:neko, "priv/rules.yml")
  # ...
end
```

### endpoint

- `:server` option

  1. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html>
  2. <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/10>

  > Runtime configuration
  >
  > :server - when true, starts the web server when the endpoint supervision tree
  > starts. Defaults to false. The mix phx.server task automatically sets this to true.

  _config/prod.exs_ (see also auto-generated comment titled `Using releases`):

  ```diff
  config :billing, BillingWeb.Endpoint,
    load_from_system_env: true,
  - url: [host: "example.com", port: 80]
  + url: [host: "example.com", port: 80],
  + server: true
  ```

  if web server is not started you'll get `Connection refused` error
  when trying to send any request to application.

- `:load_from_system_env` and `:http` options

  _config/prod.exs_:

  ```elixir
  # you won't find the :http configuration below, but set inside
  # BillingWeb.Endpoint.init/2 when load_from_system_env is true.
  # Any dynamic configuration should be done there.
  # ...
  config :billing, BillingWeb.Endpoint,
    load_from_system_env: true,
    url: [host: "example.com", port: 80],
    server: true
  ```

  _lib/billing_web/endpoint.ex_:

  ```elixir
  def init(_key, config) do
    if config[:load_from_system_env] do
      port = System.get_env("PORT") || raise "expected the PORT environment variable to be set"
      {:ok, Keyword.put(config, :http, [:inet6, port: port])}
    else
      {:ok, config}
    end
  end
  ```

### migrations

1. <https://github.com/bitwalker/distillery/issues/2>
2. <http://blog.plataformatec.com.br/2016/04/running-migration-in-an-exrm-release/>
3. <http://blog.firstiwaslike.com/elixir-deployments-with-distillery-running-ecto-migrations/>
4. <https://github.com/bitwalker/distillery/blob/master/docs/Running%20Migrations.md>

NOTE: if using edeliver just run `mix edeliver migrate production up` task
      instead - it runs migrations remotely without all this hassle.
      though this module still might be useful if you're planning to add
      more release tasks to it later.

_lib/release/tasks.ex_:

```elixir
defmodule Release.Tasks do
  @app :billing
  @repo Billing.Repo

  def migrate do
    {:ok, _} = Application.ensure_all_started(@app)
    path = Application.app_dir(@app, "priv/repo/migrations")
    Ecto.Migrator.run(@repo, path, :up, all: true)
    :init.stop()
  end
end
```

on production machine:

```sh
$ bin/billing stop
$ bin/billing command Elixir.Release.Tasks migrate
```

## test production release locally

1. <https://hexdocs.pm/distillery/terminology.html>
2. <https://hexdocs.pm/distillery/walkthrough.html>

### create production database

```sh
$ psql -d postgres
=# CREATE USER billing_prod WITH PASSWORD 'billing_prod';
=# ALTER USER billing_prod CREATEDB;
$ mix ecto.setup
```

### link _prod.secret.exs_

```sh
$ sudo ln -s $PWD/config/prod.secret.exs /var/prod.secret.exs
```

### build production release

<https://hexdocs.pm/distillery/walkthrough.html#deploying-your-release>:

> The artifact you will want to deploy is the release tarball, which is
> located at `_build/prod/rel/<name>/releases/<version>/<name>.tar.gz`.

<https://hexdocs.pm/phoenix/deployment.html#putting-it-all-together>:

```sh
# no idea how it's different from `mix deps.get`
$ mix deps.get --only prod
# compiles project into _build/prod/lib/ directory
$ MIX_ENV=prod mix compile
# builds release in _build/prod/rel/ directory
$ MIX_ENV=prod mix release
```

`MIX_ENV=prod`, inter alia, sets working directory to _\_build/prod/_
for both `compile` and `release` tasks.

in all examples I've seen `MIX_ENV=prod` and `--env=prod` are used
together like this:

```sh
$ MIX_ENV=prod mix release --env=prod
```

using `MIX_ENV=prod` only:

- sets working directory to _\_build/prod/_
- applies production configuration from _rel/config.exs_

using `--env=prod` only:

- doesn't change working directory (_\_build/dev/_ directory is used)
- applies production configuration from _rel/config.exs_
- includes extra _\_build/dev/rel/billing/var/_ directory

TL;DR: for `release` task it's safe to use `MIX_ENV=prod` only.

### run production release

```sh
$ PORT=4000 _build/prod/rel/billing/bin/billing console
```

in another terminal:

```sh
$ curl -X POST -d '{"user":{"name":"Jane"}}' -H "Content-Type: application/json" http://localhost:4000/v1/users
```

## deploy

### edeliver

#### install Erlang and Elixir on build server

NOTE: this step has been automated with Chef.

1. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>

currently my build server is production one (Ubuntu 16.04.3 LTS (Xenial Xerus)).

- connect to build server as application user (`billing`):

  ```sh
  $ ssh billing
  ```

- install `build-essential` package to compile `certifi` dependency

  ```sh
  $ sudo apt-get install build-essential
  ```

- install latest Erlang and Elixir

  ```sh
  $ mkdir tmp/
  $ cd tmp/
  $ wget http://packages.erlang-solutions.com/ubuntu/erlang_solutions.asc
  $ sudo apt-key add erlang_solutions.asc
  $ echo "deb http://packages.erlang-solutions.com/ubuntu xenial contrib" >> /etc/apt/sources.list
  $ sudo apt-get update
  $ sudo apt-get -y install esl-erlang
  $ sudo apt-get -y install elixir
  ```

#### create systemd service

1. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>

NOTE: this step has been automated with Chef.

deployed and started application must be listening on specified port:

```sh
$ sudo journalctl -ef -u phoenix_billing
localhost systemd[1]: Started Phoenix server for billing app.
localhost billing[3448]: 08:52:35.970 [info] Running BillingWeb.Endpoint with Cowboy using http://:::4000
```

#### build and deploy release

NOTE: push all changes to github!!! when building new release on build
      server edeliver fetches repo from github (just like Capistrano).

1. <http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>

```sh
$ mix edeliver build release --verbose
$ mix edeliver deploy release to production --verbose
$ mix edeliver start production --verbose
$ mix edeliver migrate production up --verbose
$ mix edeliver ping production
```

locations on production machine:

- _bin/\<app\_name\>_ - main application script
- _releases/start_erl.data_ - file with current release version
  (used by main application script to determine what version to run)
- _releases/\<release\_version\>/_ - specific release

## manage application in production

- `bin/billing pid` - get pid of running application
- `bin/billing ping` - check if application is running
- `bin/billing start` - start as daemon
- `bin/billing foreground` - start in the foreground
- `bin/billing console` - start with console attached
- `bin/billing stop`
- `bin/billing restart` - restart application daemon without shutting down VM
- `bin/billing reboot` - restart application daemon with shutting down VM
- `bin/billing remote_console` - remote shell to running application console

## debug on production machine

1. <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/5>

- systemd journal

  ```sh
  $ sudo journalctl -ef -u phoenix_billing
  ```

- Erlang VM log

  ```sh
  $ tail -f var/log/erlang.log.1
  ```

- get information about endpoint

  ```sh
  $ bin/billing remote_console
  iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint.Server
  iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint
  ```

## don't use hot upgrades

<https://hackernoon.com/state-of-the-art-in-deploying-elixir-phoenix-applications-fe72a4563cd8>:

> The downside is that you need to migrate data structures in your application.
> Deployment is no longer a no-brainer (as it should be in the continuous
> deployment world).
>
> Simply restart.

<https://hexdocs.pm/distillery/walkthrough.html#building-an-upgrade-release>:

> You do not have to use hot upgrades, you can simply do rolling restarts by
> running stop, extracting the new release tarball over the top of the old,
> and running start to boot the release.

## configure to work with Nginx

<https://medium.com/@a4word/setting-up-phoenix-elixir-with-nginx-and-letsencrypt-ada9398a9b2c>:

> There are many ways to configure your Phoenix app for production use,
> but here just make sure that you bind yourself to 127.0.0.1.
>
> Now your site is being served up only through SSL (using Nginx),
> and Phoenix is no longer directly available on port 4000.

_config/prod.exs_:

```diff
config :billing, BillingWeb.Endpoint,
  load_from_system_env: true,
+ http: [ip: {127.0.0.1}],
  url: [host: "example.com", port: 80],
  server: true
```
