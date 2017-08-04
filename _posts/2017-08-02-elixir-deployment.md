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

## preparation

- <https://hexdocs.pm/phoenix/deployment.html>
- <https://hexdocs.pm/distillery/terminology.html>
- <https://hexdocs.pm/distillery/walkthrough.html>
- <https://elixirforum.com/t/elixir-deployment-tools-general-discussion-blog-posts-wiki/827?source_topic_id=2345>

### secrets

<https://hexdocs.pm/phoenix/deployment.html#handling-of-your-application-secrets>

- replace all values in _config/prod.exs_ with environment variables and set
  those variables in production machine
- hard-code secrets in _config/prod.exs_ and place it in production machine
  manually or via Chef, say, at _/var/prod.secret.exs_

  _config/prod.exs_:

  ```elixir
  - import_config "config/prod.secret.exs"
  + import_config "/var/prod.secret.exs"
  ```

  on development machine (or else you'll get errors about missing config
  when compiling project with `MIX_ENV=prod`):

  ```sh
  $ sudo ln -s $PWD/config/prod.secret.exs /var/prod.secret.exs
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

in Elixir module:

```elixir
defmodule Neko.Reader do
  @rules_path Application.app_dir(:neko, "priv/rules.yml")
  # ...
end
```

### web server

- <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/10>

<https://hexdocs.pm/phoenix/Phoenix.Endpoint.html>:

> Runtime configuration
>
> :server - when true, starts the web server when the endpoint supervision tree
> starts. Defaults to false. The mix phx.server task automatically sets this to true.

_config/prod.exs_:

```elixir
config :billing, BillingWeb.Endpoint,
  load_from_system_env: true,
- url: [host: "example.com", port: 80]
+ url: [host: "example.com", port: 80],
+ server: true
```

also see auto-generated comment `Using releases` in _config/prod.exs_.

if web server is not started you'll get `Connection refused` error.

## manual deployment

### prepare for building release

<https://hexdocs.pm/phoenix/deployment.html#putting-it-all-together>:

```sh
# no idea how it's different from `mix deps.get`
$ mix deps.get --only prod
# compiles project into _build/prod/ directory
$ MIX_ENV=prod mix compile
```

### build release

<https://hexdocs.pm/distillery/walkthrough.html#deploying-your-release>:

> The artifact you will want to deploy is the release tarball, which is
> located at `_build/prod/rel/<name>/releases/<version>/<name>.tar.gz`.

in all examples `MIX_ENV=prod` and `--env=prod` are used at the same time:

```sh
$ MIX_ENV=prod mix release --env=prod
```

without `MIX_ENV=prod` release is built into _\_build/dev/rel/_ directory.
still settings for `prod` environment from _rel/config.exs_ are applied
(say, ERTS is included) and generated release is almost identical to
production one except for extra _\_build/dev/rel/neko/var/_ directory.

without `--env=prod` release is built into _\_build/prod/rel/_ directory and
is completely identical to the one generated with both `MIX_ENV=prod` and
`--env=prod` => it seems to be safe to omit `--env=prod` option when setting
`MIX_ENV` environment variable to required environment.

TL;DR: use `MIX_ENV=prod` only - without `--env=prod`.

### hot upgrades

<https://hexdocs.pm/distillery/walkthrough.html#building-an-upgrade-release>:

> You do not have to use hot upgrades, you can simply do rolling restarts by
> running stop, extracting the new release tarball over the top of the old,
> and running start to boot the release.

## edeliver

### install Erlang and Elixir on build server

TODO: automate these steps with Chef?

<https://groups.google.com/forum/#!topic/elixir-lang-talk/zobme8NvlZ4>

NOTE: OS on my build server is Ubuntu 16.04.3 LTS (Xenial Xerus).

currently my build server is production one.

- connect to build server as application user (`billing`):

  ```sh
  $ ssh billing
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

- install `build-essential` to compile `certifi` dependency

  ```sh
  $ sudo apt-get install build-essential
  ```

### build and deploy release

<http://blog.plataformatec.com.br/2016/06/deploying-elixir-applications-with-edeliver/>


