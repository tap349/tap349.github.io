---
layout: post
title: Elixir - Deployment
date: 2017-08-02 21:38:18 +0300
access: public
categories: [elixir, phoenix, deployment, edeliver]
---

<!-- more -->

* TOC
{:toc}
<hr>

guides:

1. <https://elixirforum.com/t/elixir-deployment-tools-general-discussion-blog-posts-wiki/827>
2. <https://hackernoon.com/state-of-the-art-in-deploying-elixir-phoenix-applications-fe72a4563cd8>
3. <https://dustinfarris.gitbooks.io/phoenix-continuous-deployment/content/>
4. <https://jimmy-beaudoin.com/posts/elixir/phoenix-deployment/> (simple, distillery)
5. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>
6. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>
7. <https://habrahabr.ru/post/320096/>
8. <http://fletchermoore.me/blog/notes-on-deploying-phoenix/> (simple, edeliver)

official documentation:

1. <https://hexdocs.pm/phoenix/deployment.html>
2. <https://github.com/edeliver/edeliver/wiki/Configuration-(.deliver-config)>

NOTE: all paths on production host are specified relative to
      application directory located at _$DELIVER_TO/\<app\_name\>/_.

## configuration

all edeliver hooks:
<https://github.com/edeliver/edeliver/wiki/Run-additional-build-tasks>.

### secrets

1. <https://hexdocs.pm/phoenix/deployment.html#handling-of-your-application-secrets>

there are 2 alternative approaches to deal with secrets:

- dynamic configuration: replace all values in _config/prod.exs_ with
  environment variables and set those variables on production host

  1. <https://mfeckie.github.io/Phoenix-In-Production-With-Systemd/>

  when using this approach remove `pre_erlang_get_and_update_deps` hook in
  _.deliver/config_.

- embeding secrets into release: hard-code secrets in _config/prod.exs_ and
  place it on build host manually or via Chef, say, at _/var/prod.secret.exs_

  1. <https://github.com/edeliver/edeliver/wiki/Embed-Secrets---Credentials-into-the-Release>

  when building release edeliver links _/var/prod.secret.exs_
  into project directory as _config/prod.secret.exs_
  (see `pre_erlang_get_and_update_deps` hook in _.deliver/config_)
  and uses it to create consolidated release config (_sys.config_) =>
  if secrets or configuration change, build and deploy release again!

  or else it's possible to import _/var/prod.secret.exs_ in _config/prod.exs_
  directly:

  ```diff
  - import_config "prod.secret.exs"
  + import_config "/var/prod.secret.exs"
  ```

  and remove the hook so that distillery could generate _sys.config_
  using _/var/prod.secret.exs_ file itself instead of its symlink.

  IMO the whole idea of using a symlink in edeliver is to remove the knowledge
  of exact _prod.secret.exs_ file location on build host from project configs.

### assets

1. <https://hexdocs.pm/phoenix/deployment.html#compiling-your-application-assets>

compilation of static assets consists of 2 steps:

- building assets (say, using Brunch)
- generating digests and cache manifest for production (using `phx.digest` Mix task)

#### assets are not used

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

#### assets are used

1. <http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>
2. <https://github.com/edeliver/edeliver/wiki/Run-additional-build-tasks>

- add Brunch support if project was generated with `--no-brunch` option
  (see [Phoenix]({% post_url 2016-11-12-phoenix %}))
- compile static assets in edeliver `pre_erlang_clean_compile` hook

  _.deliver/config_:

  ```diff
  + pre_erlang_clean_compile() {
  +   status "Compiling static assets"
  +   __sync_remote "
  +     set -e
  +     cd '$BUILD_AT'
  +
  +     # install npm packages and build assets
  +     cd assets
  +     yarn install
  +     node node_modules/brunch/bin/brunch build --production
  +
  +     # generate digest and cache manifest
  +     cd ..
  +     mkdir -p priv/static
  +     APP='$APP' MIX_ENV='$TARGET_MIX_ENV' $MIX_CMD phx.digest $SILENCE
  +   "
  + }
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

#### `url` option

set host and port to be used in generated urls using `url` option.

_config/prod.exs_:

```diff
  config :billing, BillingWeb.Endpoint,
    load_from_system_env: true,
-   url: [host: "example.com", port: 80]
+   url: [host: "billing.***.com", port: 80]
```

#### `server` option

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
-   url: [host: "billing.***.com", port: 80]
+   url: [host: "billing.***.com", port: 80],
+   server: true
```

if web server is not started you'll get `Connection refused` error when
trying to send any request to application.

#### `load_from_system_env` and `http` options

- dynamic configuration (when Nginx is not used)

  `PORT` environment variable must be exported in either
  systemd service unit or _/home/billing/.profile_.

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

- static configuration (when Nginx is used as proxy server)

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
  - specify port manually since it's no longer loaded from `PORT` environment
    variable

  _config/prod.exs_:

  ```diff
    config :billing, BillingWeb.Endpoint,
  -   load_from_system_env: true,
  +   load_from_system_env: false,
  +   http: [ip: {127, 0, 0, 1}, port: 4000],
      url: [host: "billing.***.com", port: 80],
      server: true
  ```

### Erlang VM (EVM/BEAM) flags

NOTE: distillery must have been installed to make this configuration.

<http://ds.cs.ut.ee/courses/course-files/To303nis%20Pool%20.pdf>:

> The whole runtime system together with the language, interpreter, memory
> handling is called the Erlang Runtime System (ERTS), but the virtual machine
> is often referred to also as the BEAM.

1. <http://erlang.org/doc/man/erl.html>

when using distillery EVM flags are set in _rel/vm.args_
(or any other file set with `vm_args` setting for specific
environment in _rel/config.exs_).

options set there are passed as is to EVM process:

```
/home/billing/prod/billing/erts-9.0/bin/beam.smp -Bd\
  -- -root /home/billing/prod/billing\
  -progname home/billing/prod/billing/releases/0.0.1/billing.sh\
  -- -home /home/billing\
  ...
  -name billing_prod@127.0.0.1\
  -setcookie <cookie>\
  -smp auto\
  ...
```

NOTE: application must be restarted after changing EVM flags.

- `-name` vs. `-sname`

  makes ERTS (Erlang node) into a distributed node.

  1. <https://github.com/elixir-lang/elixir/issues/3955#issuecomment-156035367>
  2. <https://github.com/bitwalker/distillery/issues/159>

  it's allowed to omit hostname when using short name (`-sname`) -
  in general both variants are acceptable:

  ```sh
  -name billing_prod@127.0.0.1
  ```

  OR

  ```sh
  -sname billing_prod
  ```

  still at least one of these flags must be specified when using a
  separate _vm.args_ file - otherwise application will refuse to start:

  ```sh
  $ bin/billing foreground
  vm.args needs to have either -name or -sname parameter.
  ```

- `-setcookie`

  sets the magic cookie of the node.

  1. <https://github.com/bitwalker/distillery/issues/59#issuecomment-258928892>
  2. <https://stackoverflow.com/questions/35812774>

  <https://happi.github.io/theBeamBook/>:

  > In order to connect two nodes they need to share or know a secret
  > passphrase called a cookie (aka magic cookie).
  >
  > As long as you are running both nodes on the same machine and
  > the same user starts them they will automatically share the cookie
  > (in the file $HOME/.erlang.cookie).
  >
  > In the distributed case we need to make sure that all nodes know or
  > share the cookie. This can be done in three ways:
  >
  > - you can set the cookie used when talking to a specific node
  > - you can set the same cookie for all systems at start up with
  >   the -setcookie parameter or
  > - you can copy the file .erlang.cookie to the home directory of
  >   the user running the system on each machine

  by default cookie from automatically generated _~/.erlang.cookie_
  file is used - it can be overriden with `-setcookie` EVM flag
  (this is what is done in generated _rel/vm.args_ file).

  application creates _.erlang.cookie_ file at startup in its home
  directory if it's missing => each application would create their
  own cookie file instead of using a single _~/.erlang.cookie_ file
  when `HOME` environment variable is set to application directory
  in application service units - don't set it at all.

  when building release with distillery cookie is set in _vm.args_
  file so _.erlang.cookie_ file is effectively ignored (even though
  it's still created if missing).

### Chef

- create _/var/prod.secret.exs_ on build host

- install Erlang and Elixir on build host

  1. <https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>
  2. <https://docs.chef.io/resource_package.html>

  to update already installed packages specify new package version:

  ```ruby
  package 'elixir' do
    version '1.5.1-1'
    options '--yes'
  end
  ```

- create systemd service

  1. <https://medium.com/@zek/deploy-early-and-often-deploying-phoenix-with-edeliver-and-distillery-part-two-f361ef36aa10>
  2. <https://github.com/bitwalker/distillery/blob/master/docs/Use%20With%20systemd.md>
  3. <https://mfeckie.github.io/Phoenix-In-Production-With-Systemd/>

  deployed and started application must be listening on specified port:

  ```sh
  $ journalctl -fu billing_prod
  localhost systemd[1]: Started Phoenix server for billing app.
  localhost billing[3448]: [info] Running BillingWeb.Endpoint with Cowboy using http://:::4000
  ```

  important service unit options:

  - `WorkingDirectory=<app_dir>`

    set working directory instead of `HOME` environment variable
    to avoid creating many _.erlang.cookie_ files (which are not
    used all the same) - see description of `-setcookie` EVM flag.

  - `ExecStart=/<bin_dir>/<app_name> foreground`

    start application in the foreground rather than as a daemon
    (using `start` application command and `RemainAfterExit=yes` option) -
    in the latter case logs are written to EVM log file instead of
    systemd journal.

  - `Restart=no` (default)

    don't restart application automatically by systemd when it crashes
    (application is meant to be restarted by supervisor - not systemd).

## deployment

### install and configure distillery

1. <https://hexdocs.pm/distillery/getting-started.html>

inter alia, generate config (_rel/config.exs_):

```sh
$ mix release.init
```

default generated config will do in most cases
(unless you need to setup stage environment, for example).

### install and configure edeliver

inter alia, create config (_.deliver/config_) manually as instructed
in [README](https://github.com/edeliver/edeliver/blob/master/README.md).

### build and deploy release

NOTE: push all changes to github!!! when building new release on build
      server edeliver fetches repo from github (just like Capistrano).

1. <http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>

```sh
$ mix edeliver build release
$ mix edeliver deploy release production
$ mix edeliver ping production
```

or the same in one go:

```sh
$ mix edeliver update production
```

NOTE: edeliver build command doesn't allow to specify target envinroment -
      it's set using `--mix-env` option which has `prod` value by default:

```sh
$ mix edeliver --help
...
--mix-env=<env>   Build with custom mix env $MIX_ENV. Default is 'prod'
```

### restart application

it's necessary to restart application after deploying
(otherwise previous release will still be running).

- when systemd IS NOT used to manage application

  ```sh
  $ mix edeliver deploy production
  $ mix edeliver restart production
  ```

  OR

  ```sh
  $ mix edeliver deploy release production --start-deploy
  ```

  OR

  _.deliver/config_:

  ```diff
  + START_DEPLOY=true
  ```

- when systemd IS used to manage application

  ```sh
  $ ssh devops@billing sudo systemctl restart billing_prod
  ```

  don't restart application directly (as described above) -
  using edeliver tasks or application commands.

  when stopping application directly service unit enters
  failed state (because process exits) but doesn't become
  active again when application is started - as a result all
  logs are written to EVM log file instead of systemd journal.

  if you still stopped application manually stop it again
  using systemd command to make systemd aware of state change.

  TL;DR: if application is managed by systemd always use
  corresponding systemd service to start/stop application.

### run migrations

```sh
$ mix edeliver migrate production
```

ALWAYS use `--version` option when running `migrate production down` edeliver
task or else it will rollback all migrations (effectively deleting all data):

```sh
$ mix edeliver migrate production down --version=20170728105044
```

### add custom commands

1. <https://github.com/bitwalker/distillery/issues/2>
2. <http://blog.plataformatec.com.br/2016/04/running-migration-in-an-exrm-release/>
3. <http://blog.firstiwaslike.com/elixir-deployments-with-distillery-running-ecto-migrations/>
4. <https://github.com/bitwalker/distillery/blob/master/docs/Running%20Migrations.md>

custom commands allow, say, to run migrations directly on production host
or perform other tasks.

_lib/release/tasks.ex_:

```elixir
defmodule Release.Tasks do
  @app :billing
  @repo Billing.Repo

  def migrate do
    start_application()
    Ecto.Migrator.run(@repo, path(), :up, all: true)
    stop_application()
  end

  def rollback do
    start_application()
    Ecto.Migrator.run(@repo, path(), :down, step: 1)
    stop_application()
  end

  def start_application do
    {:ok, _} = Application.ensure_all_started(@app)
  end

  def stop_application do
    :init.stop()
  end

  defp path do
    Application.app_dir(@app, "priv/repo/migrations")
  end
```

on production host:

```sh
$ bin/billing stop
$ bin/billing command Elixir.Release.Tasks migrate
```

## management

### tasks and commands

- [local] edeliver tasks

  1. <https://hexdocs.pm/edeliver/Mix.Tasks.Edeliver.html>

- [remote] application commands

  - `$ bin/billing pid` - get pid of running application
  - `$ bin/billing ping` - check if application is running
  - `$ bin/billing start` - start as daemon
  - `$ bin/billing foreground` - start in the foreground
  - `$ bin/billing console` - start with console attached
  - `$ bin/billing stop`
  - `$ bin/billing restart` - restart application daemon without shutting down VM
  - `$ bin/billing reboot` - restart application daemon with shutting down VM
  - `$ bin/billing remote_console` - remote shell to running application console

- [remote] systemd commands

  - `$ sudo systemctl start billing_prod`
  - `$ sudo systemctl stop billing_prod`
  - `$ sudo systemctl restart billing_prod`
  - `$ sudo systemctl status billing_prod`

## locations on production host

- _bin/\<app\_name\>_ - main application script
- _releases/start\_erl.data_ - file with current release version

  used by main application script to determine what version to run

- _releases/\<release\_version\>/_ - specific release
- _releases/\<release\_version\>/sys.config_ - release config

  release config is compiled from all related configs in _config/_ directory
  (_config/config.exs_, _config/prod.exs_ and linked _config/prod.secret.exs_).

## logging

generally Elixir application log is written to EVM log file:

```sh
$ tail -f var/log/erlang.log.1
```

but since application service is managed by systemd all logs are
sent to systemd journal (as configured in systemd service unit):

```sh
$ journalctl --no-tail --since yesterday -fu billing_prod
```

NOTE: don't use `-e` and `-n` options (`-e` implies `-n1000`) -
      they cause some lines not to be printed (IDK why).

when application is started via systemd service unit:

- application IS logging to systemd journal
- application IS NOT logging to EVM log file

when application is started manually (service is stopped):

- application IS logging to EVM log file
- application IS NOT logging to systemd journal

### log level

- application log level

  NOTE: default application log level is `info`.

  change application log level to `debug` in _config/prod.exs_:

  ```diff
  - config :logger, level: :info
  + config :logger, level: :debug
  ```

- Ecto log level

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

### colorizing

- systemd journal

  1. <https://github.com/cornet/ccze>
  2. <https://unix.stackexchange.com/questions/278161/scrolling-output-with-ccze>
  3. <https://forums.meteor.com/t/better-logging-for-react-native-and-server/27961/4>

  use `ccze` package to colorize `journalctl` output (can be installed via Chef):

  ```sh
  $ journalctl --no-tail --since yesterday -fu billing_prod | ccze -A -o nolookups
  ```

  make sure to add `-A` (`--raw-ansi`) option - otherwise long lines are
  split into several newline-separated lines (instead of being just wrapped)
  which is very inconvenient when you need to copy'n'paste such a long line.

- Elixir logs

  1. <https://hexdocs.pm/logger/Logger.html>

  `:console` is `:logger` application default backend
  (used to log both Phoenix server and user messages).

  _config/config.exs_:

  ```diff
    config :logger, :console,
      format: "$time $metadata[$level] $message\n",
  -   metadata: [:request_id],
  +   metadata: [:request_id],
  +   colors: [
  +     enabled: true,
  +     debug: :cyan,
  +     info: :green,
  +     warn: :yellow,
  +     error: :red
  +   ]
  ```

### persistent systemd journal

1. <https://www.freedesktop.org/software/systemd/man/journald.conf.html>
2. <https://habrahabr.ru/company/selectel/blog/264731/>

_/etc/systemd/journald.conf_ (no changes):

```conf
[Journal]
#Storage=auto
```

with `Storage=auto` (default value) logs will be persisted on disk
if _/var/log/journal/_ directory exists - so create one with Chef.

view specific boot:

```sh
$ journalctl --list-boots
-1 e833ad1ae9f34f89b851d08b9ad55ee0 Wed 2017-08-16 19:03:28 UTC—Wed 2017-08-16 21:54:57 UTC
 0 c4ef341537734dc18235c9e8d2d7a76a Wed 2017-08-16 21:55:35 UTC—Thu 2017-08-17 18:49:56 UTC
$ journalctl -b 0
```

### parameter filtering

1. <https://hexdocs.pm/phoenix/Phoenix.Logger.html>
2. <https://docs.appsignal.com/elixir/configuration/parameter-filtering.html>

_config/prod.exs_:

```elixir
config :phoenix,
  :filter_parameters, ["password", "number", "exp_date"]
```

### formatting

1. <https://hexdocs.pm/logger/Logger.html>

_config/prod.exs_:

```elixir
# Do not include time - it's provided by systemd journal
config :logger, :console, format: "$metadata[$level] $message\n"
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

TL;DR: for `release` Mix task it's safe to use `MIX_ENV=prod` only.

### run production release

```sh
$ PORT=4000 _build/prod/rel/billing/bin/billing console
```

in another terminal:

{% raw %}
```sh
$ curl -X POST -d '{"user":{"name":"Jane"}}' -H "Content-Type: application/json" http://localhost:4000/v1/users
```
{% endraw %}
