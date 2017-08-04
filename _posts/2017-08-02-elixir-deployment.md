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

- <https://hexdocs.pm/phoenix/deployment.html>
- <https://hexdocs.pm/distillery/terminology.html>
- <https://hexdocs.pm/distillery/walkthrough.html>

## secrets

<https://hexdocs.pm/phoenix/deployment.html#handling-of-your-application-secrets>

- replace all values in _config/prod.exs_ with environment variables and set
  those variables in production machine
- hard-code secrets in _config/prod.exs_ and place it in production machine
  manually or via Chef, say, at _/var/prod.secrets.exs_

  also don't forget to import it in _config/prod.exs_ (remove existing import):

  ```elixir
  import_config "/var/prod.secrets.exs"
  ```

## assets

<https://hexdocs.pm/phoenix/deployment.html#compiling-your-application-assets>:

> If you are not serving or don’t care about assets at all, you can just remove
> the cache_static_manifest configuration from config/prod.exs.

so if your application doesn't have to deal with assets remove specified line
in _config/prod.exs_:

```diff
config :billing, BillingWeb.Endpoint,
  load_from_system_env: true,
+ url: [host: "example.com", port: 80]
- url: [host: "example.com", port: 80],
- cache_static_manifest: "priv/static/cache_manifest.json"
```

## artifacts (say, YAML files)

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

## building release for production environment

<https://hexdocs.pm/distillery/walkthrough.html#deploying-your-release>:

> The artifact you will want to deploy is the release tarball, which is
> located at `_build/prod/rel/<name>/releases/<version>/<name>.tar.gz`.

in all examples both `MIX_ENV=prod` environment variable and `--env=prod` option
are specified:

```sh
$ MIX_ENV=prod mix release --env=prod
```

without `MIX_ENV=prod` release is built in _\_build/dev/rel/_ directory.
still settings from `prod` environment in _rel/config.exs_ are applied
(ERTS is included) and generated release is almost identical to production
one except for extra _\_build/dev/rel/neko/var/_ directory.

without `--env=prod` release is built in _\_build/prod/rel/_ directory and
is completely identical to the one generated with both `MIX_ENV=prod` and
`--env=prod`. so it seems to be safe to omit `--env=prod` option when setting
`MIX_ENV` environment variable to required environment.

TL;DR: use `MIX_ENV=prod` only - without `--env=prod` option.

## hot upgrades

<https://hexdocs.pm/distillery/walkthrough.html#building-an-upgrade-release>:

> You do not have to use hot upgrades, you can simply do rolling restarts by
> running stop, extracting the new release tarball over the top of the old,
> and running start to boot the release.
