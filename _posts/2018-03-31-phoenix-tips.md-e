---
layout: post
title: Phoenix - Tips
date: 2018-03-31 14:19:58 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

(how to) make application available from outside
------------------------------------------------

say, it might be useful to make application available on local network in
development so that it can be tested by others.

```diff
  # config/config.exs

  config :billing, BillingWeb.Endpoint,
+   # bind application to local IP address
+   http: [ip: {192, 168, 0, 36}, port: 4000],
```

using SSL
---------

1. <https://hexdocs.pm/phoenix/endpoint.html#using-ssl>
2. <http://ohanhi.com/phoenix-ssl-localhost.html>
3. <https://gist.github.com/tadast/9932075>

the idea is that it's possible to generate private key and self-signed
certificate and specify them in `https` key of endpoint configuration.

umbrella apps vs. contexts
--------------------------

> <https://www.reddit.com/r/elixir/comments/6bpah8/phoenix_context_vs_elixir_umbrella_apps/dhprpl5/>
>
> The rule of thumb is to use umbrellas where you can use separate storage
> and contexts everywhere else. Many quite big apps use only one database.
> It doesn't make sense to configure them separately in Umbrellas and you
> can't stop one part without bringing entire system down. It doesn't make
> sense to extract them to umbrellas, but you still don't want to tangle
> stuff too much.

(how to) stop children of application supervisor
------------------------------------------------

children will be automatically restarted by application supervisor =>
stop application supervisor and start required supervisors manually.

### example

sometimes it might be necessary to put application into maintenance mode
when the whole application is stopped but you still need access to DB.

the point is that you cannot stop child supervisor or worker - it will be
restarted by application supervisor immediately:

```
iex> Supervisor.which_children(MyApp.Supervisor)
[
  {MyApp.TaskSupervisor, #PID<0.482.0>, :supervisor, [Task.Supervisor]},
  {MyAppWeb.Endpoint, #PID<0.468.0>, :supervisor, [MyAppWeb.Endpoint]},
  {MyApp.Repo, #PID<0.435.0>, :supervisor, [MyApp.Repo]},
  {MyApp.Scheduler, #PID<0.428.0>, :worker, [MyApp.Scheduler]}
]
iex> Supervisor.stop(:c.pid(0,428,0))
:ok
iex> Supervisor.which_children(MyApp.Supervisor)
[
  {MyApp.TaskSupervisor, #PID<0.482.0>, :supervisor, [Task.Supervisor]},
  {MyAppWeb.Endpoint, #PID<0.468.0>, :supervisor, [MyAppWeb.Endpoint]},
  {MyApp.Repo, #PID<0.435.0>, :supervisor, [MyApp.Repo]},
  {MyApp.Scheduler, #PID<0.1854.0>, :worker, [MyApp.Scheduler]}
]
```

=> stop application supervisor instead and start `MyApp.Repo` supervisor
manually afterwards (this is what I do in _test/test_helper.exs_ - start
required supervisors only without starting the whole application):

```
iex> Supervisor.stop(MyApp.Superrvisor)
:ok
[info] Application my_app exited: normal
iex> MyApp.Repo.start_link()
{:ok, #PID<0.486.0>}
```

***UPDATE***

unfortunately this method doesn't work in production:

```
iex(my_app@127.0.0.1)2> Supervisor.stop(MyApp.Supervisor)
:ok
iex(my_app@127.0.0.1)3> *** ERROR: Shell process terminated! (^G to start new job) ***
```

=> I cannot start a new shell in production like in development.

(how to) perform signature verification
---------------------------------------

1. <https://www.userinterfacing.com/verifying-request-signatures-in-elixir-phoenix>
2. <https://github.com/hamiltop/ashes/blob/master/lib/ashes/github_webhook_plug.ex>

see [Phoenix - Troubleshooting]({% post_url 2018-12-28-phoenix-troubleshooting %})
post on how to read request body in custom plug.

```elixir
# lib/my_app_web/router.ex

defmodule MyAppWeb.Router do
  use MyAppWeb, :router

  pipeline :line do
    plug MyAppWeb.Plugs.Line.SignatureVerification
  end
end
```

```elixir
# lib/my_app_web/plugs/line/signature_verification.ex

defmodule MyAppWeb.Plugs.Line.SignatureVerification do
  import Plug.Conn
  import Plug.Crypto, only: [secure_compare: 2]

  @channel_secret Application.get_env(:my_app, :line)[:channel_secret]

  def init(opts), do: opts

  def call(conn, _opts) do
    my_signature = signature(conn)
    [line_signature] = get_req_header(conn, "x-line-signature")

    if secure_compare(my_signature, line_signature) do
      conn
    else
      unauthorized(conn)
    end
  end

  defp signature(conn) do
    body = MyAppWeb.CacheBodyReader.read_cached_body(conn)

    :sha256
    |> :crypto.hmac(@channel_secret, body)
    |> Base.encode64()
  end

  defp unauthorized(conn) do
    conn
    |> send_resp(401, "Unauthorized")
    |> halt()
  end
end
```
