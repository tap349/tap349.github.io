---
layout: post
title: Elixir - Deployment with Bootleg
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

1. <https://hexdocs.pm/bootleg/reference/role_host_options.html>

```sh
$ mix bootleg.init
```

```elixir
# config/deploy.exs

use Bootleg.DSL

config :host, Application.get_env(:lucy, LucyWeb.Endpoint)[:url][:host]
config :user, "lucy"
config :build_path, "/tmp/bootleg/build"
# release archive is unpacked right inside this directory
# (subdirectory with application name is not created)
config :deploy_path, "/home/lucy/prod/app"
config :release_path, "/home/lucy/prod/releases"
config :silently_accept_hosts, true

# `build` role defines what remote server release should be built on
role(
  :build,
  config(:host),
  # SSH username
  user: config(:user),
  # Path of the remote build workspace (:build role) or application
  # workspace (:app role)
  workspace: config(:build_path),
  # For :build roles, this is the path where the newly-built release
  # should be copied.
  # For :app roles, this is the path where the release should be found.
  # You probably want to use the same value for both!
  release_workspace: config(:release_path),
  silently_accept_hosts: config(:silently_accept_hosts)
)
```

tips
----

### change release version

by default project version from _mix.exs_ is used - or else it can be overriden
in Bootleg config:

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

### run migrations

1. <https://hexdocs.pm/distillery/guides/running_migrations.html>
2. <https://dockyard.com/blog/2018/08/23/announcing-distillery-2-0>

```sh
# rel/commands/migrate.sh

#!/bin/sh

release_ctl eval --mfa "Lucy.ReleaseTasks.migrate/0"
```

```diff
  # config/deploy/production.exs

+ task :migrate do
+   remote :db do
+     "bin/lucy migrate"
+   end
+ end

  before_task(:compile, :symlink_secret_file)
+ after_task(:deploy, :migrate)
```

### run migrations manually

I did it once when I accidentally modified old migration and wanted to run all
migrations starting from that one again. in fact it was the first migration so
I just dropped all tables including `schema_migrations` one in `psql` and run
`Reika.ReleaseTasks.migrate()` in IEx:

```
$ bin/my_app remote_console
iex> Reika.ReleaseTasks.migrate()
```

or else run custom `migrate` command:

```
$ bin/my_app migrate
```

the gotcha is that Phoenix application stops (IDK why) after running migrations
this way so make sure to start/restart it afterwards:

```sh
$ sudo systemctl restart my_app_prod
```

### compile assets

1. <https://hexdocs.pm/phoenix/deployment.html>
2. <https://hexdocs.pm/bootleg/reference/phoenix.html>

```diff
  # config/deploy/production.exs

+ task :phx_digest do
+   remote :build, cd: "assets" do
+     "npm install"
+     "npm run deploy"
+   end
+
+   remote :build do
+     "MIX_ENV=prod mix phx.digest"
+   end
+ end

  task :migrate do
    remote :db do
      "bin/lucy migrate"
    end
  end

  before_task(:compile, :symlink_secret_file)
+ after_task(:compile, :phx_digest)
  after_task(:deploy, :migrate)
```

`npm run deploy` runs `deploy` script from _assets/package.json_:

```json
"scripts": {
  "deploy": "webpack --mode production"
}
```

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

> <https://www.reddit.com/r/node/comments/83omh4/best_approaches_to_setting_up_environment/dvm2zo7>
>
> Even for packages that you use in multiple places, you can install them
> separately in each project's local node_modules. To easily run them, you can
> use npx if you're using the npm client: `$ npx webpack`. I believe npx comes
> bundled with more recent versions of npm. If you're using Yarn instead, you
> can do `$ yarn webpack`, which should find Webpack in the local node_modules.

so you can use either npm or Yarn to install npm packages and run Webpack but
Yarn is not natively supported by Phoenix:

> <https://github.com/phoenixframework/phoenix/pull/1963#issuecomment-396079993>
>
> npm has had improvements as well since yarn's release, and users are free
> as always to use yarn on their new projects themselves.

Mix tasks use npm under the hood => stick to npm in Bootleg tasks as well.

another important reason to choose npm is that _assets/package-lock.json_
is not used by Yarn when installing npm packages during deployment - it
looks for _assets/yarn.lock_ (which is missing for obvious reasons since
Mix tasks don't use Yarn).
