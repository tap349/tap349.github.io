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

- <https://hexdocs.pm/distillery/terminology.html>
- <https://hexdocs.pm/distillery/walkthrough.html>

## including data files in release

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

TL;DR: use `MIX_ENV=prod` only without `--env=prod` option.
