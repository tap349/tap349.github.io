---
layout: post
title: Elixir - Stage Environment
date: 2017-08-13 18:34:14 +0300
access: public
categories: [elixir, phoenix, deployment]
---

<!-- more -->

* TOC
{:toc}
<hr>

NOTE: everywhere except for edeliver, environments have short names
      (`prod`/`stage`) including Phoenix application itself, Chef,
      names of secret files, Nginx sites and systemd service units.

## configuration

### Chef

create based on current environment:

- Nginx site (_billing_stage_ or _billing_prod_)
- secret file (_/var/stage.secret.exs_ or _/var/prod.secret.exs_)
- systemd service unit (_billing_stage.service_ or _billing_prod.service_)

add for `stage` environment:

- bash aliases
- PostgreSQL user and database

### configs

NOTE: these settings must be synchronized with Chef.

_config/stage.secret.exs_:

- specify stage database credentials

_config/stage.exs_:

- specify different port (say, 4001)
- `import_config "stage.secret.exs"`

### distillery

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
-   set cookie: :"foo123"
  end
```

_rel/vm.args.stage_:

```elixir
## Name of the node
-name billing_stage.0.0.1

## Cookie for distributed erlang
## (generate with `mix phoenix.gen.secret`)
-setcookie foo123

# Enable SMP automatically based on availability
-smp auto
```

_rel/vm.args.prod_:

```elixir
## Name of the node
-name billing_prod@127.0.0.1

## Cookie for distributed erlang
## (generate with `mix phoenix.gen.secret`)
-setcookie foo123

# Enable SMP automatically based on availability
-smp auto
```

The Little Elixir and OTP Guidebook:

> think of nodes as separate Erlang runtimes that can talk to each other.

### edeliver

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

## deployment

```sh
$ mix edeliver build release --mix-env=stage
$ mix edeliver deploy release to staging
$ mix edeliver migrate staging up
$ mix edeliver ping staging
```

make sure that application is restarted -
see instructions for restarting production release.

## alternative solutions

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
