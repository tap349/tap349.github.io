---
layout: post
title: AppSignal - Tips
date: 2019-01-16 11:50:55 +0300
access: public
comments: true
categories: [appsignal, elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) notify AppSignal of a new deploy
-----------------------------------------

1. <https://docs.appsignal.com/application/markers/deploy-markers.html#config-option>

add `revision` config option:

```diff
  # config/appsignal.exs

  config :appsignal, :config,
    active: true,
-   name: "MyApp"
+   name: "MyApp",
+   revision: Mix.Project.config()[:version]
```

NOTE: don't forget to update project version in _mix.exs_.

deploying release with the same version has no effect:

- new deploy marker is not created
- the last deploy is not modified (its deploy date stays the same)

(how to) add Absinthe integration
---------------------------------

1. <https://docs.appsignal.com/elixir/integrations/absinthe.html>

```elixir
# lib/my_app_web/plugs/appsignal_absinthe_plug.ex

defmodule MyAppWeb.Plugs.AppsignalAbsinthePlug do
  def init(opts), do: opts

  @path "/api"
  def call(%Plug.Conn{request_path: @path, method: "POST"} = conn, _opts) do
    Appsignal.Transaction.set_action("POST " <> @path)
    conn
  end

  def call(conn, _), do: conn
end
```

```elixir
# lib/my_app_web/router.ex

defmodule MyAppWeb.Router do
  use MyAppWeb, :router

  pipeline :api do
    plug :accepts, ["json"]
    plug MyAppWeb.Plugs.AppsignalAbsinthePlug
  end

  # ...
end
```

use tags instead of specific metric name prefixes
-------------------------------------------------

> <https://docs.appsignal.com/metrics/custom.html>
>
> Custom metrics sometimes need some context what they're about. This
> context can be added as tags so it doesn't need to be included in the
> name and you can use the same metric name for different values.
>
> We do not recommend adding this context to your metric names like so:
> eu.database_size, us.database_size and asia.database_size. This creates
> multiple metrics that serve the same purpose. The same goes for any
> dynamic string that builds the metric key, e.g. user_#{user.id}.

```elixir
# bad
Appsignal.set_gauge("fb.stat.certainty", value)

# good
Appsignal.set_gauge("stat.certainty", value, %{platform: "fb"})
```
