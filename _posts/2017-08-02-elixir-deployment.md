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
4. <https://jimmy-beaudoin.com/posts/elixir/phoenix-deployment/> (simple, distillery)
5. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>
6. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>
7. <https://habrahabr.ru/post/320096/>
8. <http://fletchermoore.me/blog/notes-on-deploying-phoenix/> (simple, edeliver)

NOTE: all paths on production host are specified relative to
      application directory located at _$DELIVER_TO/\<app\_name\>/_.

## preparing for deployment

1. <https://hexdocs.pm/phoenix/deployment.html>

### secrets

1. <https://hexdocs.pm/phoenix/deployment.html#handling-of-your-application-secrets>

- replace all values in _config/prod.exs_ with environment variables and
  set those variables on production host (dynamic configuration) OR
- hard-code secrets in _config/prod.exs_ and place it on production host
  manually or via Chef, say, at _/var/prod.secret.exs_

  _config/prod.exs_:

  ```diff
  - import_config "prod.secret.exs"
  + import_config "/var/prod.secret.exs"
  ```

  NOTE: it's not necessary to change `import_config` line when using
        edeliver - it automatically links _/var/prod.secret.exs_ into
        project directory as _config/prod.secret.exs_ when building release.

### assets

<https://hexdocs.pm/phoenix/deployment.html#compiling-your-application-assets>:

> If you are not serving or don’t care about assets at all, you can just remove
> the cache_static_manifest configuration from config/prod.exs.

so if application doesn't deal with assets remove this line in _config/prod.exs_:

```diff
  config :billing, BillingWeb.Endpoint,
    load_from_system_env: true,
+   url: [host: "example.com", port: 80]
-   url: [host: "example.com", port: 80],
-   cache_static_manifest: "priv/static/cache_manifest.json"
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
  -   url: [host: "example.com", port: 80]
  +   url: [host: "example.com", port: 80],
  +   server: true
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

on production host:

```sh
$ bin/billing stop
$ bin/billing command Elixir.Release.Tasks migrate
```

## testing production release locally

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

## deployment

### edeliver

#### configure and run Chef

- install Erlang and Elixir on production host

  1. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>

- create systemd service

  1. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>

  deployed and started application must be listening on specified port:

  ```sh
  $ sudo journalctl -ef -u billing_prod
  localhost systemd[1]: Started Phoenix server for billing app.
  localhost billing[3448]: 08:52:35.970 [info] Running BillingWeb.Endpoint with Cowboy using http://:::4000
  ```

#### embedding secrets into release

1. <https://github.com/edeliver/edeliver/wiki/Embed-Secrets---Credentials-into-the-Release>

when building release on remote host edeliver links _/var/prod.secret.exs_
into its build directory as _config/prod.secret.exs_
(see `pre_erlang_get_and_update_deps` hook in _.deliver/config_)
and uses it to create consolidated release config (_sys.config_) =>
if secrets or configuration change, build and deploy release again!

or else it's possible import _/var/prod.secret.exs_ in _config/prod.exs_
directly and remove the hook so that distillery could generate _sys.config_
using _/var/prod.secret.exs_ file itself instead of its symlink.

IMO the whole idea of using a symlink in edeliver is to remove the knowledge
of exact _prod.secret.exs_ file location on build host from project configs.

#### build and deploy release

NOTE: push all changes to github!!! when building new release on build
      server edeliver fetches repo from github (just like Capistrano).

1. <http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>
2. <https://github.com/edeliver/edeliver/wiki/Configuration-(.deliver-config)>

```sh
$ mix edeliver build release
$ mix edeliver deploy release to production
$ mix edeliver migrate production up
$ mix edeliver ping production
```

make sure to restart application after deploying
(otherwise previous release will still be running):

- run `restart` task right after deploying

  ```sh
  $ mix edeliver restart production
  ```

- pass `--start-deploy` option to `deploy` task

  ```sh
  $ mix edeliver deploy release to production --start-deploy
  ```

- add `START_DEPLOY=true` to edeliver config

  _.deliver/config_:

  ```diff
  + START_DEPLOY=true
  ```

#### locations on production host

- _bin/\<app\_name\>_ - main application script
- _releases/start_erl.data_ - file with current release version

  used by main application script to determine what version to run

- _releases/\<release\_version\>/_ - specific release
- _releases/\<release\_version\>/sys.config_ - release config

  release config is compiled from all related configs in _config/_ directory
  (_config/config.exs_, _config/prod.exs_ and linked _config/prod.secret.exs_).

## managing

- application commands

  - `bin/billing pid` - get pid of running application
  - `bin/billing ping` - check if application is running
  - `bin/billing start` - start as daemon
  - `bin/billing foreground` - start in the foreground
  - `bin/billing console` - start with console attached
  - `bin/billing stop`
  - `bin/billing restart` - restart application daemon without shutting down VM
  - `bin/billing reboot` - restart application daemon with shutting down VM
  - `bin/billing remote_console` - remote shell to running application console

- systemd commands

  - `sudo systemctl start billing_prod`
  - `sudo systemctl stop billing_prod`
  - `sudo systemctl restart billing_prod`

## debugging

1. <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/5>

- get information about endpoint

  ```sh
  $ bin/billing remote_console
  iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint.Server
  iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint
  ```

## logging

generally Elixir application log is written to Erlang VM log file:

```sh
$ tail -f var/log/erlang.log.1
```

but since application service is managed by systemd all logs are
sent to systemd journal (as configured in systemd service unit):

```sh
$ journalctl -ef -u billing_prod
```

when application is started via systemd service unit:

- application IS logging to systemd journal
- application IS NOT logging to Erlang VM log file

when application is started manually (service is stopped):

- application IS logging to Erlang VM log file
- application IS NOT logging to systemd journal

### application log level

NOTE: default application log level is `info`.

change application log level to `debug` in _config/prod.exs_:

```diff
- config :logger, level: :info
+ config :logger, level: :debug
```

### Ecto log level

1. <https://hexdocs.pm/ecto/Ecto.Repo.html>
2. <https://stackoverflow.com/a/30319577/3632318>

NOTE: default Ecto log level is `info`.

change Ecto log level to `debug` in _config/prod.secret.exs_
(don't forget to synchronize Chef application cookbook):

```diff
  config :billing, Billing.Repo,
    adapter: Ecto.Adapters.Postgres,
    username: "username",
    password: "password",
    database: "database",
-   pool_size: 15
+   pool_size: 15,
+   loggers: [{Ecto.LogEntry, :log, [:debug]}]
```

it's required because some Ecto errors have `debug` log level, say:

```
[debug] ** (Ecto.Query.CastError) deps/ecto/lib/ecto/repo/queryable.ex:331
```

## hot upgrades

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

## using proxy server (Nginx)

1. <https://hexdocs.pm/phoenix/phoenix_behind_proxy.html>
2. <https://medium.com/@a4word/setting-up-phoenix-elixir-with-nginx-and-letsencrypt-ada9398a9b2c>

> There are many ways to configure your Phoenix app for production use,
> but here just make sure that you bind yourself to 127.0.0.1.
>
> Now your site is being served up only through SSL (using Nginx),
> and Phoenix is no longer directly available on port 4000.

- disable dynamic endpoint configuration (`load_from_system_env: false`) -
  or else you cannot specify any `http` options in _config/prod.exs_
  (they get overriden by dynamic configuration in _lib/billing_web/endpoint.ex_)
- bind application to `127.0.0.1` so that it's available to Nginx only
  (and not available from outside accordingly)
- specify port manually because it's no longer loaded from `PORT` environment
  variable
- set host and port to be used in generated urls using `url` option

_config/prod.exs_:

```diff
  config :billing, BillingWeb.Endpoint,
-   load_from_system_env: true,
-   url: [host: "example.com", port: 80],
+   load_from_system_env: false,
+   http: [ip: {127, 0, 0, 1}, port: 4000],
+   url: [host: "billing.***.com", port: 80],
    server: true
```

## stage environment

NOTE: everywhere except for edeliver environments have short names
      (`prod`/`stage`) including Phoenix application itself, Chef,
      names of secret files, Nginx sites and systemd service units.

### configure and run Chef

create based on current environment:

- Nginx site (_billing_stage_ or _billing_prod_)
- secret file (_/var/stage.secret.exs_ or _/var/prod.secret.exs_)
- systemd service unit (_billing_stage.service_ or _billing_prod.service_)

add for `stage` environment:

- bash aliases
- PostgreSQL user and database

### create configs for _stage_ environment

NOTE: these settings must be synchronized with Chef.

_config/stage.secret.exs_:

- specify stage database credentials

_config/stage.exs_:

- specify different port (say, 4001)
- `import_config "stage.secret.exs"`

### configure edeliver

_.deliver/config_:

```diff
- DELIVER_TO="/home/billing/stage"
+ TEST_AT="/home/billing/stage"

  pre_erlang_get_and_update_deps() {
+   local _secret_file="$TARGET_MIX_ENV.secret.exs"
+   __sync_remote "
+     ln -sfn "/var/$_secret_file" "$BUILD_AT/config/$_secret_file"
+   "
  }
```

### configure distillery

1. <https://hexdocs.pm/distillery/runtime-configuration.html#content>
2. <https://github.com/bitwalker/distillery/issues/159> (!)
3. <https://github.com/bitwalker/distillery/issues/121>
4. <https://stackoverflow.com/questions/33406725>

_rel/config.exs_:

```diff
+ environment :stage do
+   set vm_args: "rel/vm.args.stage"
+   set include_erts: true
+   set include_src: true
+ end

  environment :prod do
+   set vm_args: "rel/vm.args.prod"
    set include_erts: true
    set include_src: false
-   set cookie: :"123"
  end
```

_rel/vm.args.stage_:

```elixir
## Name of the node
-name billing_stage.0.0.1

## Cookie for distributed erlang
## (generate with `mix phoenix.gen.secret`)
-setcookie 123

# Enable SMP automatically based on availability
-smp auto
```

_rel/vm.args.prod_:

```elixir
## Name of the node
-name billing_prod@127.0.0.1

## Cookie for distributed erlang
## (generate with `mix phoenix.gen.secret`)
-setcookie 123

# Enable SMP automatically based on availability
-smp auto
```

### build and deploy release

```sh
$ mix edeliver build release --mix-env=stage
$ mix edeliver deploy release to staging
$ mix edeliver migrate staging up
$ mix edeliver ping staging
```

make sure that application is restarted -
just like after deploying production release.

### alternative solutions

<https://stackoverflow.com/questions/38818446>:

> :prod is in fact a build mode and should be used for any cases where one intends
> to deploy. So in other words, my staging deployment should be set to MIX_ENV=prod
> and then either use environment variables for dynamic configuration settings in
> the prod.exs file or, as I have done in this case, dynamically load a deployment
> specific configuration in prod.exs like so:

> deployment_config=System.get_env("DEPLOYMENT_CONFIG")
> import_config "./deployment_config/#{deployment_config}.exs"

idea looks brilliant but this solution works only if you have separate
staging and production hosts (which is not my case).
