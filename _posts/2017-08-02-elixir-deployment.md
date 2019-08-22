---
layout: post
title: Elixir - Deployment
date: 2017-08-02 21:38:18 +0300
access: public
comments: true
categories: [elixir, phoenix, deployment]
---

<!-- more -->

<!-- prettier-ignore -->
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

configuration
-------------

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

  and remove the hook so that Distillery could generate _sys.config_
  using _/var/prod.secret.exs_ file itself instead of its symlink.

  IMO the whole idea of using a symlink in edeliver is to remove the knowledge
  of exact _prod.secret.exs_ file location on build host from project configs.

### assets

1. <https://hexdocs.pm/phoenix/deployment.html#compiling-your-application-assets>

compilation of static assets consists of 2 steps:

- building assets (say, using Brunch or Webpack)
- generating digests and cache manifest for production (using `phx.digest` Mix
  task)

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
  +     npm install
  +     node node_modules/brunch/bin/brunch build --production
  +
  +     # generate digest and cache manifest
  +     cd ..
  +     mkdir -p priv/static
  +     # remove &> /dev/null at the end - or else we
  +     # don't see actual errors if phx.digest task fails
  +     #APP='$APP' MIX_ENV='$TARGET_MIX_ENV' $MIX_CMD phx.digest $SILENCE
  +     APP='$APP' MIX_ENV='$TARGET_MIX_ENV' $MIX_CMD phx.digest
  +   "
  + }
  ```

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

### artifacts (resources, resource files - say, YAML or JSON files)

<https://elixirforum.com/t/including-data-files-in-a-distillery-release/2813>:

> The traditional place to put non-code resources that are needed at runtime
> is the priv folder. All the tools are aware of this convention and preserve
> proper paths.

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

2 ways to get _priv/_ directory itself (or the file inside it) at runtime:

- `Application.app_dir(:billing, "priv/foo.txt")`
- `Path.join(:code.priv_dir(:billing), "foo.txt")`

for example:

```elixir
defmodule Neko.Reader do
  @rules_path Application.app_dir(:neko, "priv/rules.yml")
  # ...
end
```

however both ways don't work in _config/config.exs_ because application is
not started yet (and hence unknown) when the former is being compiled - in
this case just reference _priv/_ directory directly using relative path:

```elixir
# config/config.exs

config :neko, :rules, dir: "priv/rules"
```

this should work in most cases. however _priv/_ directory wasn't found
using relative path when I tried to access it from inside release task
(see <https://hexdocs.pm/distillery/guides/running_migrations.html>).
I had to update application environment value in the task itself:

```elixir
new_rules_config =
  :neko
  |> Application.get_env(:rules)
  |> update_in([:dir], &Application.app_dir(:neko, &1))

Application.put_env(:neko, :rules, new_rules_config)
```

### endpoint

#### `url` option

set host and port to be used in generated URLs using `url` option.

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
> starts. Defaults to false. The mix phx.server task automatically sets this to
> true.

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

  1. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
  2. <https://hexdocs.pm/phoenix/phoenix_behind_proxy.html>
  3. <https://medium.com/@a4word/setting-up-phoenix-elixir-with-nginx-and-letsencrypt-ada9398a9b2c>

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
  - start web server when endpoint supervision tree starts

  _config/prod.exs_:

  ```diff
    config :billing, BillingWeb.Endpoint,
  -   load_from_system_env: true,
  -   http: [:inet6, port: System.get_env("PORT") || 4000],
  -   url: [host: "example.com", port: 80],
  +   load_from_system_env: false,
  +   http: [ip: {127, 0, 0, 1}, port: 4000],
  +   url: [host: "billing.***.com", port: 80],
  +   server: true
  ```

  it's possible to omit `ip: {127, 0, 0, 1}` (it must be used by default):

  _config/prod.exs_:

  ```elixir
  config :billing, BillingWeb.Endpoint,
    # ...
    http: [port: 4000],
    # ...
  ```

  add `:inet6` option if it's necessary to listen on IPv6 as well:

  ```elixir
  config :billing, BillingWeb.Endpoint,
    # ...
    http: [:inet6, port: 4000],
    # ...
  ```

### Distillery

default generated config will do in most cases (unless you need to set up
stage environment, for example).

#### Distillery config (rel/config.exs)

- `include_erts` environment setting

  1. <https://github.com/bitwalker/distillery/blob/master/docs/introduction/walkthrough.md#configuring-your-release>

  > <https://github.com/bitwalker/distillery/blob/master/docs/config/distillery.md#environmentrelease-settings>
  >
  > include_erts - whether to include the system ERTS or not

  in general it's recommended to bundle ERTS so that "the target system does
  not need to have Erlang or Elixir installed". in my case though release is
  built on and deployed to the same host so it makes sense to use system-wide
  ERTS and not include it into release - it helps to reduce its final size by
  almost 10 MB:

  ```
  $ ls -l releases
  28551477 Apr  5 19:45 0.1.0.tar.gz
  19013521 Apr  6 00:21 0.2.0.tar.gz
  ```

  ***UPDATE (2019-04-07)***

  if ERTS is not bundled, `MyApp.ReleaseTasks` module used by Distillery to
  run migration doesn't see Ecto repos:

  ```elixir
  Application.get_env(:my_app, :ecto_repos, [])
  # => []
  ```

  => set `include_erts` setting to `true` as recommended.

  ***UPDATE (2019-04-10)***

  moreover if ERTS is not bundled, old versions of dependencies are loaded in
  production for some reason: each release archive contains _releases/RELEASES_
  file where all dependencies are listed with their paths and versions.

  release archive is extracted to the same location every time - chances are
  this location already has _lib/_ directory created by previous releases and
  contains old (older) versions of dependencies. so with time _lib/_ directory
  accumulates many different versions of dependencies.

  long story short, with `include_erts: false` setting the oldest version of
  each dependency from _lib/_ is used which causes weird errors with undefined
  functions, etc.

  => set `include_erts` setting to `true` as recommended. again.

#### EVM config (rel/vm.args)

1. <http://erlang.org/doc/man/erl.html>

EVM = Erlang VM (aka BEAM).

<http://ds.cs.ut.ee/courses/course-files/To303nis%20Pool%20.pdf>:

> The whole runtime system together with the language, interpreter, memory
> handling is called the Erlang Runtime System (ERTS), but the virtual machine
> is often referred to also as the BEAM.

when using Distillery EVM flags are set in _rel/vm.args_ (or any other file
set with `vm_args` setting for specific environment in _rel/config.exs_).

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

  turns Erlang node into a distributed node.

  1. <https://github.com/elixir-lang/elixir/issues/3955#issuecomment-156035367>
  2. <https://github.com/bitwalker/distillery/issues/159>

  omit hostname when using short name (`-sname`) - in general both variants are
  acceptable:

  ```sh
  -name billing_prod@127.0.0.1
  ```

  OR

  ```sh
  -sname billing_prod
  ```

  still at least one of these flags must be specified when using a separate
  _vm.args_ file - otherwise application will refuse to start:

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
  > As long as you are running both nodes on the same machine and the same
  > user starts them they will automatically share the cookie (in the file
  > $HOME/.erlang.cookie).
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

  when building release with Distillery cookie is set in _vm.args_
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

  - `WorkingDirectory=<APP_DIR>`

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

deployment
----------

### install and configure edeliver

inter alia, create config (_.deliver/config_) manually as instructed
in [README](https://github.com/edeliver/edeliver/blob/master/README.md).

#### auto-versioning

1. <https://github.com/edeliver/edeliver/wiki/Auto-Versioning>

auto-versioning allows to append version metadata to release version
(it's equal to application version by default as configured in
_rel/config.exs_ while application version is set in _mix.exs_):

_.deliver/config_:

```sh
AUTO_VERSION=build-date+git-branch+git-revision
```

resulting release version (example): `neko_0.2.0+20180527-master-6bfed7e`
(`neko` is application name here).

NOTE: application version must be incremented manually in _mix.exs_.

### build and deploy release

NOTE: push all changes to github!!! when building new release on build
      server edeliver fetches repo from github (just like Capistrano).

1. <http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>

```sh
$ mix edeliver build release
$ mix edeliver deploy release production
```

or the same in one go:

```sh
$ mix edeliver update production
```

NOTE: edeliver `build` command doesn't allow to specify target envinroment -
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

  don't restart application directly - using edeliver tasks
  (as described above) or application commands.

  when stopping application directly service unit enters
  failed state (because process exits) but doesn't become
  active again when application is started - as a result all
  logs are written to EVM log file instead of systemd journal.

  if you still stopped application manually stop it again
  using systemd command to make systemd aware of state change.

  TL;DR: if application is managed by systemd always use
  corresponding systemd service to start/stop application.

### run migrations

1. <https://github.com/edeliver/edeliver/issues/81>

NOTE: migrations should be run after restarting application -
      otherwise new release is not loaded yet and migrations
      are not seen (they will be shown as pending afterwards).

```sh
$ mix edeliver migrate production
```

ALWAYS use `--version` option when running edeliver `migrate production down`
command or else it will rollback all migrations (effectively deleting all data):

```sh
$ mix edeliver migrate production down --version=20170728105044
```

or else just don't use `change/0` functions in migrations -
only `up/0` and `down/0` ones making migrations irreversible.

### ping node and check release version

```sh
$ mix edeliver ping production
$ mix edeliver version production
```

### rollback release

NOTE: migrations should be rollbacked before restarting application -
      they just might be not available when previous realese is loaded.

```sh
$ mix edeliver deploy release production --version=<previous_release_version>
$ mix edeliver migrate production down --version=<previous_migration_version>
$ ssh devops@billing sudo systemctl restart billing_prod
```

release version is defined as `AUTO_VERSION` in _.deliver/config_
(see `auto-versioning` section above).

tasks and commands
------------------

### [local] edeliver tasks

1. <https://hexdocs.pm/edeliver/Mix.Tasks.Edeliver.html>

on tasks vs. commands: in wiki and `mix edeliver --help` tasks and
commands are used interchangeably but to be precise `edeliver` is
a Mix task itself so `edeliver build release` is a edeliver build
task as well while `build release` is a specific edeliver command
(edeliver `build release` command vs. `edeliver build release` task).

but for simplicity I might refer to edeliver commands as tasks as well.

### [remote] application commands

`console`, `foreground`, `start` - application boot commands.

- `$ bin/billing pid` - get pid of running application
- `$ bin/billing ping` - check if application is running
- `$ bin/billing start` - start as daemon
- `$ bin/billing foreground` - start in the foreground
- `$ bin/billing console` - start with console attached
- `$ bin/billing stop`
- `$ bin/billing restart` - restart application daemon without shutting down VM
- `$ bin/billing reboot` - restart application daemon with shutting down VM
- `$ bin/billing remote_console` - remote shell to running application console

### [remote] systemd commands

- `$ sudo systemctl start billing_prod`
- `$ sudo systemctl stop billing_prod`
- `$ sudo systemctl restart billing_prod`
- `$ sudo systemctl status billing_prod`

### [remote] custom commands

1. <https://hexdocs.pm/distillery/guides/running_migrations.html>
2. <https://dockyard.com/blog/2018/08/23/announcing-distillery-2-0>
3. <http://blog.plataformatec.com.br/2016/04/running-migration-in-an-exrm-release/>

custom commands allow, say, to run migrations or build ES indexes directly on
production host:

```elixir
# lib/reika/release_tasks.ex

# https://hexdocs.pm/mix/Mix.Tasks.Release.html#module-one-off-commands-eval-and-rpc
# https://hexdocs.pm/distillery/guides/running_migrations.html
defmodule Reika.ReleaseTasks do
  @start_apps [
    :crypto,
    :ssl,
    :postgrex,
    :ecto_sql,
    :elasticsearch
  ]

  @app :reika
  @repos Application.get_env(@app, :ecto_repos, [])
  @es_cluster Reika.ES.Cluster
  @es_indexes [:reika_shops]

  def migrate do
    start_apps()
    start_repos()

    IO.puts("Running migrations...")
    Enum.each(@repos, &run_migrations_for/1)

    stop_apps()
  end

  # https://hexdocs.pm/elasticsearch/distillery.html
  def build_es_indexes do
    start_apps()
    start_repos()
    start_es_cluster()

    IO.puts("Building ES indexes...")

    Enum.each(@es_indexes, fn es_index ->
      new_es_config =
        @app
        |> Application.get_env(@es_cluster)
        |> update_in(
          [:indexes, es_index, :settings],
          &Application.app_dir(@app, &1)
        )

      Application.put_env(@app, @es_cluster, new_es_config)

      # restart ES Cluster so that configuration is re-read
      GenServer.stop(@es_cluster)
      start_es_cluster()

      Elasticsearch.Index.hot_swap(@es_cluster, es_index)
    end)

    stop_apps()
  end

  # -----------------------------------------------------------------
  # migrations
  # -----------------------------------------------------------------

  defp run_migrations_for(repo) do
    migrations_path = priv_path_for(repo, "migrations")
    Ecto.Migrator.run(repo, migrations_path, :up, all: true)
  end

  defp priv_path_for(repo, filename) do
    app = Keyword.get(repo.config, :otp_app)

    repo_underscore =
      repo
      |> Module.split()
      |> List.last()
      |> Macro.underscore()

    priv_dir = "#{:code.priv_dir(app)}"
    Path.join([priv_dir, repo_underscore, filename])
  end

  # -----------------------------------------------------------------
  # start/stop helpers
  # -----------------------------------------------------------------

  defp start_apps do
    IO.puts("Loading #{@app}..")
    # Load the code for @app, but don't start it
    Application.load(@app)

    IO.puts("Starting apps..")
    Enum.each(@start_apps, &Application.ensure_all_started/1)
  end

  defp start_repos do
    IO.puts("Starting repos..")
    # > Ecto requires a pool size of at least 2 to support
    # > concurrent migrators.
    # > When migrations run, Ecto uses one connection to
    # > maintain a lock and another to run migrations.
    Enum.each(@repos, & &1.start_link(pool_size: 2))
  end

  defp start_es_cluster do
    IO.puts("Starting ES cluster..")
    @es_cluster.start_link()
  end

  defp stop_apps do
    IO.puts("Success!")
    :init.stop()
  end
end
```

on production host:

```sh
$ bin/reika stop
$ bin/reika command Elixir.Reika.ReleaseTasks migrate
```

locations on production host
----------------------------

- _bin/\<app\_name\>_ - main application script
- _releases/start\_erl.data_ - file with current release version

  used by main application script to determine what version to run

- _releases/\<release\_version\>/_ - specific release
- _releases/\<release\_version\>/sys.config_ - release config

  release config is compiled from all related configs in _config/_ directory
  (_config/config.exs_, _config/prod.exs_ and linked _config/prod.secret.exs_).

logging
-------

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

hot upgrades
------------

> <https://hackernoon.com/state-of-the-art-in-deploying-elixir-phoenix-applications-fe72a4563cd8>
>
> The downside is that you need to migrate data structures in your application.
> Deployment is no longer a no-brainer (as it should be in the continuous
> deployment world).
>
> Simply restart.

> <https://hexdocs.pm/distillery/walkthrough.html#building-an-upgrade-release>
>
> You do not have to use hot upgrades, you can simply do rolling restarts by
> running stop, extracting the new release tarball over the top of the old,
> and running start to boot the release.

testing production release locally
----------------------------------

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

1. <https://hexdocs.pm/distillery/introduction/installation.html#your-first-release>

> <https://hexdocs.pm/distillery/walkthrough.html#deploying-your-release>
>
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

by default Distillery uses release environment which matches the value of
`MIX_ENV` (that is `Mix.env()`):

```elixir
# rel/config.exs

use Mix.Releases.Config,
  # ...
  # This sets the default environment used by `mix release`
  default_environment: Mix.env()
```

but it's possible to use another release environment with `--env` flag:
in this case release will be compiled and built using specified release
environment and current Mix environment simultaneously:

```sh
# both configurations are used:
#
# - `staging` release environment configuration from rel/config.ex
# - `prod` Mix environment configuration from config/prod.ex
$ MIX_ENV=prod mix release --env=staging
```

Mix environment also uses release environment to determine the location
where project should be compiled and built - say, it's _\_build/prod/_ in
case of `MIX_ENV=prod`.

in all examples I've seen `MIX_ENV=prod` and `--env=prod` are used together
but it's sufficient to use `MIX_ENV=prod` only because in this case release
environment is automatically set to `prod` in _rel/config.exs_.

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

tips
----

### rerun all migrations in production

I did it once when I accidentally modified old migration and wanted to run all
migrations starting from that one again. in fact it was the first migration so
I just dropped all tables including `schema_migrations` one in `psql` and run
`Reika.ReleaseTasks.migrate()` in IEx:

```
$ bin/billing remote_console
iex> Reika.ReleaseTasks.migrate()
```

or else run custom `migrate` command:

```
$ bin/billing migrate
```

the gotcha is that Phoenix application stops (IDK why) after running migrations
this way so make sure to start/restart it afterwards:

```sh
$ sudo systemctl restart billing_prod
```

troubleshooting
---------------

### dependency is not included in distillery release

`phoenix_expug` package has `expug` dependency but it's not added to
distillery release (raising error at runtime):

```sh
$ mix release --verbose
...
=> One or more direct or transitive dependencies are missing from
    :applications or :included_applications, they will not be included
    in the release:

    :expug
    :parse_trans

    This can cause your application to fail at runtime. If you are sure
    that this is not an issue, you may ignore this warning.
```

systemd journal:

```
Request: GET /admin/transfers
** (exit) an exception was raised:
    ** (UndefinedFunctionError) function Expug.Runtime.attr/2 is undefined (module Expug.Runtime is not available)
        Expug.Runtime.attr("lang", "en")
        (billing) lib/billing_web/templates/layout/admin.html.pug:2: BillingWeb.LayoutView."admin.html"/1
```

**solution**

1. <https://github.com/hashrocket/gatling/issues/24#issuecomment-270044265>
2. <https://github.com/bitwalker/distillery/issues/55>

probably this is because `phoenix_expug` uses `applications` option in
_mix.exs_ (`application` callback) where all dependencies that should
be started are listed explicitly. it's deprecated now and overrides new
default behaviour when all dependencies from `deps` option (`project`
callback) are added to `applications` implicitly.

while `iex phx.server` seems to start all dependencies listed both in
`applications` and `deps`, distillery apparently does add `applications`
only to release treating `deps` as compile-time dependencies.

one solution is to remove `applications` altogether so that all `deps`
are added to `applications` by default though it's not an option when
dealing with external dependencies (unless you fork them).

another solution is to add those missing depedencies (from the ouput of
running `mix release --verbose` command) to _rel/config.exs_:

```diff
  release :billing do
    set version: current_version(:billing)
    set applications: [
-     :runtime_tools
+     :runtime_tools,
+     expug: :load
    ]
  end
```

### logs are truncated in systemd journal

say, we have a very long `PARes` XML field that used to be truncated
in systemd journal all the time.

**solution**

log message was truncated by `Kernel.inspect/2` - not by systemd journal.
solution is not to use the former (where it's possible of course - that
is when argument is a string):

```diff
- Logger.info("API REQUEST:\n" <> inspect(soap))
+ Logger.info("API REQUEST:\n" <> soap)
```

NOTE: long request parameter values are still truncated by Phoenix -
      IDK how to change this behaviour.

### systemd service is restarted twice

I restart application systemd service after each deploy:

```elixir
# mix.exs

defp deploy(_) do
  Mix.Task.run("bootleg.build")
  Mix.Task.run("bootleg.deploy")

  Mix.Task.run(
    :cmd,
    ["ssh devops@XXX.XXX.XXX.XX sudo systemctl restart my_app_prod"]
  )
end
```

the problem is that the service is restarted twice: right after it's stopped
and started for the first time it's getting stopped again for some reason.

systemd journal:

```
21:46:26 systemd[1]: Stopping my_app service (prod)...
21:46:28 my_app[23151]: ok
21:46:38 systemd[1]: Stopped my_app service (prod).
21:46:38 systemd[1]: Started my_app service (prod).
21:46:40 systemd[1]: Stopping my_app service (prod)...
21:46:43 my_app[23623]: Node my_app@127.0.0.1 is not running!
21:46:43 systemd[1]: my_app_prod.service: Control process exited, code=exited status=1
21:46:44 my_app[23391]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:init] started
21:46:44 my_app[23391]: module=Phoenix.Endpoint.Cowboy2Adapter [info] Running MyAppWeb.Endpoint with cowboy 2.6.3 at 0.0.0.0:4000 (http)
21:46:44 my_app[23391]: module=Phoenix.Endpoint.Supervisor [info] Access MyAppWeb.Endpoint at http://XXX.XXX.XXX.XX
21:46:49 my_app[23391]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:cluster_wait] joining cluster..
21:46:49 my_app[23391]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:cluster_wait] no connected nodes, proceeding without sync
21:48:13 systemd[1]: my_app_prod.service: State 'stop-sigterm' timed out. Killing.
21:48:13 systemd[1]: my_app_prod.service: Killing process 23391 (beam.smp) with signal SIGKILL.
21:48:13 systemd[1]: my_app_prod.service: Killing process 23543 (erl_child_setup) with signal SIGKILL.
21:48:13 systemd[1]: my_app_prod.service: Killing process 23773 (inet_gethost) with signal SIGKILL.
21:48:13 systemd[1]: my_app_prod.service: Killing process 23774 (inet_gethost) with signal SIGKILL.
21:48:13 systemd[1]: my_app_prod.service: Killing process 23807 (appsignal-agent) with signal SIGKILL.
21:48:13 systemd[1]: my_app_prod.service: Main process exited, code=killed, status=9/KILL
21:48:13 systemd[1]: my_app_prod.service: Failed with result 'exit-code'.
21:48:13 systemd[1]: Stopped my_app service (prod).
21:48:13 systemd[1]: Started my_app service (prod).
21:48:17 my_app[23823]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:init] started
21:48:17 my_app[23823]: module=Phoenix.Endpoint.Cowboy2Adapter [info] Running MyAppWeb.Endpoint with cowboy 2.6.3 at 0.0.0.0:4000 (http)
21:48:17 my_app[23823]: module=Phoenix.Endpoint.Supervisor [info] Access MyAppWeb.Endpoint at http://XXX.XXX.XXX.XX
21:48:22 my_app[23823]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:cluster_wait] joining cluster..
21:48:22 my_app[23823]: module=Swarm.Logger [info] [swarm on my_app@127.0.0.1] [tracker:cluster_wait] no connected nodes, proceeding without sync
```

it looks like `Mix.Task.run/2` tries to rerun specified task if the latter
times out - that is if no response is received within a set amount of time.

**solution**

use `:os.cmd/1` to execute a shell command instead:

```elixir
# mix.exs

defp deploy(_) do
  Mix.Task.run("bootleg.build")
  Mix.Task.run("bootleg.deploy")

  :os.cmd('ssh devops@XXX.XXX.XXX.XX sudo systemctl restart my_app_prod')
end
```
