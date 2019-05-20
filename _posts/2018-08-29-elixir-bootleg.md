---
layout: post
title: Elixir - Bootleg
date: 2018-08-29 12:06:25 +0300
access: public
comments: true
categories: [elixir, phoenix, deployment]
---

<!-- more -->

* TOC
{:toc}
<hr>

configuration
-------------

```sh
$ mix bootleg.init
```

full listing of _config/deploy.exs_ and _config/deploy/production.exs_
with comments:

```elixir
# config/deploy.exs

use Bootleg.DSL

# map Bootleg environment to Mix environment
mix_env =
  case config(:env) do
    # by default when not specified at all
    :production -> "prod"
    # when specified explicitly as a string
    # in mix.exs or passed on command line
    "production" -> "prod"
    "staging" -> "stage"
    env -> "Unknown environment: #{inspect(env)}"
  end

# https://hexdocs.pm/bootleg/config/environments.html
# https://hexdocs.pm/bootleg/reference/options.html
#
# overrides Mix environment inside Bootleg - it doesn't change MIX_ENV
# so other services (like CircleCI) continue to work as usual.
#
# according to docs default value is "prod" but it's not set anywhere -
# this option is always read as `config({:mix_env, "prod"})` effectively
# using "prod" by default when the value is nil.
#
# still I need to know current Mix environment to build different paths
# so I use current Bootleg environment to set `mix_env` option which is
# then used to build paths.
#
# NOTE: when configured this way Bootleg is fully agnostic about MIX_ENV.
config :mix_env, mix_env

# https://hexdocs.pm/bootleg/reference/role_host_options.html
# https://hexdocs.pm/bootleg/reference/options.html
config :app, :my_app
config :version, Mix.Project.config()[:version]
# SSH username
config :user, "#{config(:app)}"
config :build_path, "/tmp/bootleg/build"
# release archive is unpacked right inside this directory
# (subdirectory with application name is not created)
config :deploy_path, "/home/#{config(:app)}/#{config(:mix_env)}/app"
config :release_path, "/home/#{config(:app)}/#{config(:mix_env)}/releases"
config :silently_accept_hosts, true

task :symlink_secret_file do
  remote :build do
    """
    ln -sfn \
      /var/#{config(:app)}/config/#{config(:mix_env)}.secret.exs \
      #{config(:build_path)}/config/#{config(:mix_env)}.secret.exs
    """
  end
end

# https://hexdocs.pm/phoenix/deployment.html
# https://hexdocs.pm/bootleg/reference/phoenix.html
task :phx_digest do
  remote :build, cd: "assets" do
    "npm install"
    # runs `deploy` script from assets/package.json
    "npm run deploy"
  end

  remote :build do
    "MIX_ENV=prod mix phx.digest"
  end
end

# https://hexdocs.pm/distillery/guides/running_migrations.html
# https://dockyard.com/blog/2018/08/23/announcing-distillery-2-0
task :migrate do
  remote :db do
    # runs rel/commands/migrate.sh script
    "bin/#{config(:app)} migrate"
  end
end

task :restart_service do
  service_name = "#{config(:app)}_#{config(:mix_env)}"

  remote :system do
    "sudo systemctl restart #{service_name}"
  end
end

# https://hexdocs.pm/bootleg/reference/workflow.html
before_task(:compile, :symlink_secret_file)
after_task(:compile, :phx_digest)
after_task(:deploy, :migrate)
after_task(:migrate, :restart_service)
```

```elixir
# config/deploy/production.exs

use Bootleg.DSL

config :host, "172.XXX.XXX.XX"

# `build` role defines what remote server release should be built on
role(
  :build,
  config(:host),
  user: config(:user),
  # path of the remote build workspace
  workspace: config(:build_path),
  # path where the newly-built release should be copied
  release_workspace: config(:release_path),
  silently_accept_hosts: config(:silently_accept_hosts)
)

# `app` role defines what remote servers release should be deployed to
role(
  :app,
  config(:host),
  user: config(:user),
  # path of application workspace
  workspace: config(:deploy_path),
  # path where the release should be found
  # (use the same value for build and app roles)
  release_workspace: config(:release_path),
  silently_accept_hosts: config(:silently_accept_hosts)
)

# https://hexdocs.pm/bootleg/config/roles.html
#
# custom roles
role(:db, config(:host), user: config(:user), workspace: config(:deploy_path))
role(:system, config(:host), user: "devops")
```

running
-------

```elixir
# mix.exs

defp aliases do
  [
    # ...
    deploy: &deploy/1
  ]
end

defp deploy(_) do
  Mix.Task.run("bootleg.build", ["production"])
  Mix.Task.run("bootleg.deploy", ["production"])
end
```

notes
-----

### compiling assets

#### Bootleg 0.10.0

make sure the version of Bootleg is > 0.10.0 because there is no dedicated
`remote_generate_release` step in Bootleg 0.10.0 - currently it's available
in master only:

```diff
  # https://github.com/labzero/bootleg/blob/master/docs/reference/workflow.md

  Remote Builds

  - compile
+ - remote_generate_release
  - release_workspace set?
    - yes
      - copy_build_release
    - no
      - download_release
```

that is in Bootleg 0.10.0 `compile` task both compiles project and builds
release using Distillery => `phx_digest` hook is executed when release has
been already built => compiled assets are not included into release.

#### Yarn vs. npm

1. <https://medium.com/@vincentnewkirk/npm-vs-yarn-2019-e88757b17038>

> <https://www.reddit.com/r/node/comments/83omh4/best_approaches_to_setting_up_environment/dvm2zo7>
>
> Even for packages that you use in multiple places, you can install them
> separately in each project's local node_modules. To easily run them, you can
> use npx if you're using the npm client: `$ npx webpack`. I believe npx comes
> bundled with more recent versions of npm. If you're using Yarn instead, you
> can do `$ yarn webpack`, which should find Webpack in the local node_modules.

so you can use either npm or Yarn to install npm packages and run Webpack -
they are now pretty close in terms of features and speed.

- support by Phoenix

  1. <https://github.com/phoenixframework/phoenix/pull/1963>
  2. <https://github.com/yarnpkg/yarn/issues/5654>

  Yarn is not natively supported by Phoenix:

  > <https://github.com/phoenixframework/phoenix/pull/1963#issuecomment-396079993>
  >
  > It would be an implicit lookup and another thing to go wrong outside of the
  > team's direct control. npm has had improvements as well since yarn's release,
  > and users are free as always to use yarn on their new projects themselves.

  "not natively supported" means if you generate new Phoenix project and opt to
  fetch and install dependencies, npm is used to install node modules:

  ```
  $ mix phx.new foo
  ...
  Fetch and install dependencies? [Yn]
  * running mix deps.get
  * running cd assets && npm install && node node_modules/webpack/bin/webpack.js --mode development
  * running mix deps.compile
  ```

  as a result _package-lock.json_ is generated while Yarn uses _yarn.lock_ as
  a lockfile and doesn't recognize _package-lock.json_.

  this might cause problems if you use npm in development and Yarn to compile
  assets for production since different package versions may get installed in
  the end.

  => always make sure to use the same package manager in both development and
  production if you prefer not to stick to npm all the time.

- speed

  ```
  $ mix phx.new foo
  $ cd foo/assets

  $ rm -rf node_modules; date; npm install; date
  Mon May 20 13:46:33 MSK 2019
  ...
  Mon May 20 13:47:01 MSK 2019

  $ rm -rf node_modules; date; yarn install; date
  Mon May 20 13:47:29 MSK 2019
  ...
  Mon May 20 13:48:10 MSK 2019
  ```

  so npm even runs 10s faster than Yarn - this doesn't say that npm is faster
  than Yarn but it says that Yarn is not several times faster than npm like it
  used to be in the very beginning.

=> stick to npm in Bootleg tasks.

tips
----

### change Git branch

1. <https://hexdocs.pm/bootleg/reference/options.html>

```elixir
# config/deploy.exs

config :refspec, "feature/2-refactor-eva-for-anna"
```

`refspec` option is set to `master` by default.

### change release version

by default project version from _mix.exs_ is used - it can be overriden in
Bootleg config:

```elixir
# config/deploy.exs

config :version, "0.0.1"
```

NOTE: version is used in both `bootleg.build` and `bootleg.deploy` tasks!

=> `bootleg.deploy` task uses version to determine which release to deploy.

### rollback release

set previous release version:

```diff
  # config/deploy.exs

- config :version, "0.0.2"
+ config :version, "0.0.1"
```

deploy release:

```sh
$ mix bootleg.deploy
```

TODO: it's also necessary to rollback migrations to specific version -
      create corresponding release task.

troubleshooting
---------------

### SSHKit returned an internal error

```
$ MIX_ENV=prod mix bootleg.deploy
...
Copying release archive from release workspace
[172.104.236.23] (export BOOTLEG_ENV="production" REPLACE_OS_VARS="true" && /usr/bin/env mkdir -p /home/gertruda/prod/app)
** (SSHError) SSHKit returned an internal error on 172.104.236.23: {:badmatch, {:error, :badarg}}
    lib/bootleg/ssh.ex:99: anonymous fn/2 in Bootleg.SSH.run/2
    (elixir) lib/enum.ex:1327: Enum."-map/2-lists^map/1-0-"/2
    lib/bootleg/ssh.ex:118: Bootleg.SSH.run!/2
    lib/bootleg/ssh.ex:141: Bootleg.SSH.validate_workspace/3
    lib/bootleg/ssh.ex:58: Bootleg.SSH.init/2
    deps/bootleg/lib/bootleg/tasks/deploy.exs:25: anonymous fn/3 in Bootleg.DynamicTasks.CopyDeployRelease.execute/0
    (elixir) lib/enum.ex:1940: Enum."-reduce/3-lists^foldl/2-0-"/3
    lib/bootleg/dsl.ex:339: Bootleg.DSL.invoke/1
```

**solution**

always specify user name as a string in Bootleg config:

```diff
  # config/deploy.exs

- config :user, :gertruda
+ config :user, "gertruda"
```
