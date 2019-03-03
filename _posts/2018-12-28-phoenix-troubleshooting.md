---
layout: post
title: Phoenix - Troubleshooting
date: 2018-12-28 22:01:44 +0300
access: public
comments: true
categories: [elixir, phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

The task "phx.new" could not be found
-------------------------------------

```
$ mix phx.new billing --no-brunch
** (Mix) The task "phx.new" could not be found
```

**solution**

I guess this problem is related to using `asdf` to manage Elixir versions -
Phoenix has not been installed yet into new Elixir directory:

```
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
Are you sure you want to install "https://github.com/phoenixframework/archives/raw/master/phx_new.ez"? [Yn]
* creating /Users/tap/.asdf/installs/elixir/1.5.1/.mix/archives/phx_new
```

all styles are gone
-------------------

**solution**

I removed _priv/static/_ directory some time ago - restore it from Git repo
or copy from newly generated project:

```sh
$ mix phx.new hello --no-brunch
```

connect() failed (111: Connection refused) while connecting to upstream
-----------------------------------------------------------------------

_/home/reika/prod/nginx/log/error.log_:

```
2018/12/18 11:17:27 [error] 4829#4829: *14 connect() failed (111: Connection refused)
  while connecting to upstream, client: 109.195.XXX.XXX, server: 139.162.XXX.XXX,
  request: "GET /hello HTTP/1.1", upstream: "http://127.0.0.1:4000/hello", host:
  "139.162.XXX.XXX"
```

**solution**

1. [Elixir - Deployment]({% post_url 2017-08-02-elixir-deployment %})

most likely Cowboy is not started:

> <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
>
> :server - when true, starts the web server when the endpoint supervision tree
>   starts. Defaults to false. The mix phx.server task automatically sets this
>   to true

```diff
  # config/prod.exs

  config :reika, ReikaWeb.Endpoint,
-   load_from_system_env: true,
-   http: [:inet6, port: System.get_env("PORT") || 4000],
-   url: [host: "example.com", port: 80],
+   load_from_system_env: false,
+   http: [port: 4000],
+   url: [host: "139.162.XXX.XXX", port: 80],
+   server: true
```

corresponding line will appear in systemd journal once Cowboy is started
(it will be missing if enpdoint is not configured as described above):

```
reika[15112]: 11:36:06.017 [info] [swarm on reika@127.0.0.1] [tracker:init] started
reika[15112]: 11:36:06.320 [info] Running ReikaWeb.Endpoint with cowboy 2.6.1 at http://139.162.XXX.XXX
reika[15112]: 11:36:11.022 [info] [swarm on reika@127.0.0.1] [tracker:cluster_wait] joining cluster..
```

request body is empty
---------------------

1. <https://github.com/phoenixframework/phoenix/issues/459>
2. <https://github.com/elixir-plug/plug/issues/691>

the problem is that request body is discarded after it's read and parsed in
`Plug.Parsers` plug so it's empty in all subsequent plugs:

> <https://github.com/phoenixframework/phoenix/issues/459#issue-47825763>
>
> I need to a way to access the body of a request as a raw string. Plug
> supports this functionality but as the docs say this can only be done
> once. Once the body is consumed, trying to read the body will result
> in an empty string.

but request body is usually required, say, to perform signature verification.

**solution**

1. <https://hexdocs.pm/plug/Plug.Parsers.html#module-custom-body-reader>

solution added in Plug 1.5.1 is to use custom body reader in `Plug.Parsers`
plug - that reader will cache raw body so that it can be used in subsequent
plugs:

```elixir
# lib/my_app_web/cache_body_reader.ex

defmodule MyAppWeb.CacheBodyReader do
  def read_body(conn, opts) do
    {:ok, body, conn} = Plug.Conn.read_body(conn, opts)
    conn = update_in(conn.assigns[:raw_body], &[body | (&1 || [])])
    {:ok, body, conn}
  end

  def read_cached_body(conn) do
    conn.assigns[:raw_body]
  end
end
```

```diff
  # lib/my_app_web/endpoint.ex

  plug Plug.Parsers,
    parsers: [:urlencoded, :multipart, :json],
    pass: ["*/*"],
+   body_reader: {MyAppWeb.CacheBodyReader, :read_body, []},
    json_decoder: Phoenix.json_library()
```

custom plug to perform signature verification:

```diff
  # lib/my_app_web/plugs/line/signature_verification.ex

  defmodule MyAppWeb.Plugs.Line.SignatureVerification do
    defp signature(conn) do
-     {:ok, body, conn} = Plug.Conn.read_body(conn)
+     body = MyAppWeb.CacheBodyReader.read_cached_body(conn)

      # ...
    end
  end
```
