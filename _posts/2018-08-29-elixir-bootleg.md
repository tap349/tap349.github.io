---
layout: post
title: Elixir - Bootleg
date: 2018-08-29 12:06:25 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

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

TODO: it's also necessary to rollback migrations to specific version - create
      corresponding release task.

### compile assets

1. <https://hexdocs.pm/phoenix/deployment.html>
2. <https://hexdocs.pm/bootleg/reference/phoenix.html>

> <https://www.reddit.com/r/node/comments/83omh4/best_approaches_to_setting_up_environment/dvm2zo7>
>
> Even for packages that you use in multiple places, you can install them
> separately in each project's local node_modules. To easily run them, you can
> use npx if you're using the npm client: `$ npx webpack`. I believe npx comes
> bundled with more recent versions of npm. If you're using Yarn instead, you
> can do `$ yarn webpack`, which should find Webpack in the local node_modules.

```diff
  # config/deploy/production.exs

+ task :phx_digest do
+   remote :build, cd: "assets" do
+     "yarn install"
+     "yarn deploy"
+   end
+
+   remote :build do
+     "MIX_ENV=prod mix phx.digest"
+   end
+ end

  task :migrate do
    remote :db do
      "bin/eva migrate"
    end
  end

  before_task(:compile, :symlink_secret_file)
+ after_task(:compile, :phx_digest)
  after_task(:deploy, :migrate)
```

`yarn deploy` runs `deploy` script from _assets/package.json_:

```json
"scripts": {
  "deploy": "webpack --mode production"
}
```

### run migrations

I did it once when I accidentally modified old migration and wanted to run all
migrations starting from that one again. in fact it was the first migration so
I just dropped all tables including `schema_migrations` one in `psql` and run
`Reika.ReleaseTasks.migrate()` in IEx:

```
$ bin/my_app remote_console
iex> Reika.ReleaseTasks.migrate()
```

or if _rel/commands/migrate.sh_ is created:

```
$ bin/my_app migrate
```

the gotcha is that Phoenix application stops (IDK why) after running migrations
this way so make sure to start/restart it afterwards:

```sh
$ sudo systemctl restart my_app_prod
```
