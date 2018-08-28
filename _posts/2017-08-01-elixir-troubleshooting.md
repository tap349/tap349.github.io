---
layout: post
title: Elixir - Troubleshooting
date: 2017-08-01 17:48:52 +0300
access: public
comments: true
categories: [elixir, phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

(FunctionClauseError) no function clause matching in Plug.Conn.put_resp_header/3
--------------------------------------------------------------------------------

1. <https://github.com/ueberauth/guardian/issues/196>

response header value must be a string - convert `exp` variable to string:

```elixir
conn
|> put_resp_header("authorization", "Bearer #{jwt}")
|> put_resp_header("x-expires", "#{exp}")
# ...
```

Slogan: Kernel pid terminated (application_controller)
------------------------------------------------------

_/home/billing/prod/billing/erl_crash.dump_ on production host:

{% raw %}
```
Slogan: Kernel pid terminated (application_controller) ({application_start_failure,billing,{{shutdown,{failed_to_start_child,'Elixir.BillingWeb.Endpoint',{#{'__exception__' => true,'__struct__' => 'Elixir.Run
```
{% endraw %}

**solution**

in most cases it means that `PORT` environment variable is not set - examine
`~/.profile` and `/etc/systemd/system/phoenix_billing.service` files.

Node is not running! / Node not responding to pings.
----------------------------------------------------

restarting either `prod` or `stage` application leads to another
one being unreachable (sometimes restarting `prod` application
causes `stage` one to be unreachable - sometimes vice versa).

running application stops responding to ping/start/stop commands
(issued with edeliver task locally or application command on remote host).

```sh
$ ssh billing
$ prod && bin/billing ping
Node 'billing_prod@127.0.0.1' not responding to pings.
$ prod && bin/billing remote_console
Node billing_prod@127.0.0.1 is not running!
```

**solution**

application is not responding because its Erlang node is no
longer registered in epmd - see `troubleshooting` section of
[Elixir - epmd]({% post_url 2017-09-02-elixir-epmd %})
for explanation of how this could happen and how to fix it.

[Phoenix] The task "phx.new" could not be found
-----------------------------------------------

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

[Phoenix] all styles are gone
-----------------------------

**solution**

I removed _priv/static/_ directory some time ago - restore it from Git repo
or copy from newly generated project:

```sh
$ mix phx.new hello --no-brunch
```

[edeliver] Host key verification failed
---------------------------------------

```
$ mix deploy.stage
EDELIVER BILLING WITH UPDATE COMMAND
-----> Updating to revision dec86cd from branch master
-----> Building the release for the update
-----> Authorizing hosts
-----> Ensuring hosts are ready to accept git pushes
-----> Pushing new commits with git to: billing@billing
-----> Resetting remote hosts to dec86cdd704a1e88fd5fd0600bad4a425c81762e
-----> Cleaning generated files from last build
-----> Symlinking stage.secret.exs
-----> Fetching / Updating dependencies
using mix to fetch and update deps
rebar and rebar3 for mix was built already
* creating /home/billing/.mix/archives/hex-0.17.0
* Getting slime (git@github.com:slime-lang/slime.git)
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
** (Mix) Command "git --git-dir=.git fetch --force --quiet --progress" failed
```

**solution**

1. <https://github.com/docker-library/golang/issues/148#issuecomment-279459932>

use HTTPS instead of SSH to clone public repos in _mix.exs_:

```elixir
defp deps do
  [
    # ...
-   {:slime, git: "git@github.com:slime-lang/slime.git", override: true},
+   {:slime, git: "https://github.com/slime-lang/slime.git", override: true},
    # ...
  ]
end
```

[distillery] "$@" -- "${1+$ARGS}"
---------------------------------

systemd journal:

```
localhost systemd[1]: Started Phoenix server for billing.
localhost billing[1234]: "$@" -- "${1+$ARGS}"
```

**solution**

<https://github.com/bitwalker/distillery/issues/319#issuecomment-326661509>:

> It's put there when reattaching stdout to the tty after running post-start
> hooks, so it's normal. I'll see if I can silence it, but it's expected.

erl_child_setup closed
----------------------

sometimes Erlang crash dump is generated when stopping application.

_erl_crash.dump_:

```
=erl_crash_dump:0.3
Sun Sep  3 22:49:34 2017
Slogan: erl_child_setup closed
```

**solution**

1. <https://elixirforum.com/t/phoenix-server-crashing-randomly/3687>

<https://elixirforum.com/t/phoenix-server-crashing-randomly/3687/2>:

> Perhaps it has something to do with NIFs? You are using
> elixir_make/comeonin/guardian which I believe all use NIFs.

<https://elixirforum.com/t/phoenix-server-crashing-randomly/3687/14>:

> By default Erlang will launch one scheduler thread per core and 10
> "async threads" to do blocking things like IO.
>
> Additionally, more modern versions, will fork a process called
> "erl_child_setup" that is used for spawning ports to avoid forking
> the main VM process, which may be costly.

all in all it looks like forked process `erl_child_setup` is not correctly
terminated when application is stopped - maybe it has something to do with
NIFs used by Guardian package.

however this crash doesn't affect application in any way so I guess it can
be safely ignored.

(ArgumentError) argument error
------------------------------

errors occurs when trying to insert into ETS table:

```
** (exit) exited in: GenServer.call(#PID<0.465.0>, {:update, #Function<4.3141730/1 in Neko.UserRate.Store.reload/2>}, 5000)
    ** (EXIT) an exception was raised:
        ** (ArgumentError) argument error
        (stdlib) :ets.insert(:user_anime_ids, {#PID<0.465.0>, #MapSet<[]>})
```

**solution**

<https://stackoverflow.com/a/26216656/3632318>:

> By default, ETS tables are created with protected access. That means
> that any process can read values from the table, but only the process
> that created the table can write values to it.

I tried to write to ETS table inside agent while it was created
outside of it in agent wrapper - problem was solved by creating ETS
table inside anonymous function passed to `Agent.start_link/2`.

alternatively it's possible to create ETS table with `public` option so
that any process can write to it - not only its owner (of course if it's
safe to do in your application).

(MatchError) no match of right hand side value (:undef)
-------------------------------------------------------

```
** (Mix) Could not start application neko: Neko.Application.start(:normal, []) returned an error: shutdown: failed to start child: :simple_rule_worker_pool
    ** (EXIT) an exception was raised:
        ** (MatchError) no match of right hand side value: {:error, {:EXIT, {:undef, [{Neko.Rules.SimpleRule.Worker, :start_link, [[]], []}, ...]}}}
```

**solution**

application failed to start because `start_link` function of
GenServer mentioned (`Neko.Rules.SimpleRule.Worker`) had zero
arity while it must have arity 1 if it's meant to be supervised
(see [Elixir - OTP]({% post_url 2017-05-28-elixir-otp %})).

so the error can be fixed by passing dummy initial argument:

```elixir
def start_link(state \\ []) do
  GenServer.start_link(__MODULE__, state)
end
```

(EXIT) no process: the process is not alive
-------------------------------------------

getting agent value (in `Neko.UserRate.Store.all/1`) fails since agent is
no longer alive:

```
** (stop) exited in: GenServer.call(#PID<0.24355.22>, {:get, #Function<0.105777791/1 in Neko.UserRate.Store.all/1>}, 5000)
    ** (EXIT) no process: the process is not alive or there's no process currently associated with the given name, possibly because its application isn't started
    (elixir) lib/gen_server.ex:774: GenServer.call/3
    (neko) lib/neko/rules/simple_rule.ex:43: Neko.Rules.SimpleRule.achievements/2
    (neko) lib/neko/rules/simple_rule/worker.ex:34: Neko.Rules.SimpleRule.Worker.handle_call/3
    (stdlib) gen_server.erl:636: :gen_server.try_handle_call/4
    (stdlib) gen_server.erl:665: :gen_server.handle_msg/6
    (stdlib) proc_lib.erl:247: :proc_lib.init_p_do_apply/3
```

common application-specific conditions under which this error occurs:

- `action` of incoming request is always `reset`
- user handler process for `user_id` of incoming request
  has already been started when request is received

**solution**

error must be caused by this sequence of steps:

- request to reset user rates (`reset` action) is received
- agent that stores user rates is stopped synchronously

  `:DOWN` message is sent *asynchronously* - the process that monitors
  this agent (user rate store registry) will receive it eventually and
  will remove agent record (agent PID is stored under `user_id` key)
  from ETS table.

- user rates are loaded

  they are meant to be reloaded (fetched from shikimori) but it doesn't
  happen: `:DOWN` message is not received in user rate store registry yet
  => agent record is still present in ETS table => user rates are not
  reloaded because agent is still running according to the lookup in ETS
  table - and user rates are reloaded only if agent is not running
  (reloading user rates in turn starts the agent and saves its PID in
  ETS table which prevents user rates from reloading upon receiving the
  next request).

- agent value is attempted to be retrieved to calculate new achievements

  this results into error as described above: agent is not alive but
  its stale PID is still stored in ETS table since `:DOWN` message is
  not received yet in user rate store registry.

- `:DOWN` message is received eventually but it doesn't matter now

  or else `:DOWN` message may be received right after the step where user
  rates are loaded - in this case agent record will be removed from ETS
  table and retrieving agent value in the next step (where agent value is
  retrieved to calculate new achievements) will result in almost the same
  error (but PID will be `nil` instead of being stale).

  using stale PID:

  ```
  iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
  iex> Agent.stop(pid)
  iex> Agent.get(nil, &(&1))
  ** (exit) exited in: GenServer.call(#PID<0.437.0>, {:get, #Function<6.99386804/1 in :erl_eval.expr/5>}, 5000)
      ** (EXIT) no process: the process is not alive or there's no process currently associated with the given name, possibly because its application isn't started
      (elixir) lib/gen_server.ex:774: GenServer.call/3
  ```

  using `nil` as PID:

  ```
  iex> Agent.get(nil, &(&1))
  ** (exit) exited in: GenServer.call(nil, {:get, #Function<6.99386804/1 in :erl_eval.expr/5>}, 5000)
      ** (EXIT) no process: the process is not alive or there's no process currently associated with the given name, possibly because its application isn't started
      (elixir) lib/gen_server.ex:766: GenServer.call/3
  ```

to fix this problem I have to make sure that user rates are loaded
only once `:DOWN` message is received in monitoring process. IDK how
to secure this so I've decided to refuse from stopping agent at all -
now user rates are force reloaded instead (see commit a72aa5b).

(RuntimeError) load user_rate store first
-----------------------------------------

this error resembles the previous one - user rate store is not found in
user rate store registry (backed by ETS table) when I try to lookup the
former:

```elixir
defp store(user_id) do
  case Registry.lookup(user_id) do
    {:ok, store} -> store
    :error -> raise "load user_rate store first"
  end
end
```

```
$ journalctl --no-tail --since '2018-02-08 00:04:00' --until '2018-02-08 00:07:00' -u neko
...
** (RuntimeError) load user_rate store first
    (neko) lib/neko/user_rate.ex:67: Neko.UserRate.store/1
    (neko) lib/neko/user_rate.ex:61: Neko.UserRate.delete/2
    (neko) lib/neko/request.ex:28: Neko.Request.process/1
    (neko) lib/neko/user_handler.ex:55: Neko.UserHandler.handle_call/3
    (stdlib) gen_server.erl:636: :gen_server.try_handle_call/4
    (stdlib) gen_server.erl:665: :gen_server.handle_msg/6
    (stdlib) proc_lib.erl:247: :proc_lib.init_p_do_apply/3
Last message (from #PID<0.11351.59>): {:process, %Neko.Request{action: "put", id: 33277258, score: 1, status: "watching", target_id: 36840, user_id: 187700}}
```

**solution**

error must be caused by this sequence of steps:

- store (agent) crashes the caller (user handler process) because
  network request to shikimori has timed out (shikimori is down)

  agent call timeout and network request receive timeout are both
  90 seconds in my case so any of them might have happened.

  1st scenario:

  - agent call has timed out
  - the caller (task) exits

    <https://hexdocs.pm/elixir/Agent.html#get/3>:

    > If no result is received within the specified time,
    > the function call fails and the caller exits.

  - the caller that spawned the task (user handler process) exits too

    <https://hexdocs.pm/elixir/Task.html#await/2>:

    > In case the task process dies, the current process will
    > exit with the same reason as the task.

  2nd scenario:

  - network request has timed out
  - HTTPoison raises `%HTTPoison.Error{id: nil, reason: :timeout}}`
  - agent crashes (network request is performed inside it)
  - the caller (task) exits too

    TODO: WHY task process exits if it's not linked to agent?
          all synchronous calls crash the caller?

- since user rate store registry monitors all stores, it will receive
  and handle monitor `:DOWN` message sent by `Process.monitor/1` but
  it's **unknown** when `:DOWN` message will arrive since it was sent
  asynchronously

- another request for the same `user_id` is received and new user handler
  process is spawned but `:DOWN` message hasn't been received yet so user
  rates are not reloaded (reloading is skipped because store PID is still
  present in store registry even though it's stale now)

- `:DOWN` message is received and handled in user rate store registry -
  store PID (that is agent PID) is removed from ETS table

- store PID is attempted to be retrieved (say, to delete user rate from
  store) but it's no longer available which results into runtime error
  (it's raised manually when store PID is not found in store registry)

SUMMARY:

user rate store crashes in previous request but `:DOWN` message is
received while processing current request only - after loading user
rates but before using the store (error occurs because store PID is
not found in ETS table but anyway agent itself is not alive now -
it crashed in previous request).

once again IDK how to solve this problem properly - this must be a
very rare case so I've just made an error message more informative.

pacificnew: no such file or directory
-------------------------------------

```
[error] GenServer :tzdata_release_updater terminating
** (File.Error) could not stream "/home/billing/stage/billing/lib/tzdata-0.5.14/priv/tmp_downloads/339626_44358462//pacificnew": no such file or directory
```

**solution**

1. <https://github.com/lau/tzdata/issues/57>

fixed in tzdata v0.5.16.

(HTTPoison.Error) :closed
-------------------------

**solution**

1. <https://github.com/edgurgel/httpoison#note-about-broken-ssl-in-erlang-19>
2. <http://campezzi.ghost.io/httpoison-ssl-connection-closed/>

related issue in Erlang/OTP ([ERL-192](https://bugs.erlang.org/browse/ERL-192))
is resolved now so you shouldn't be receiving this error since Erlang/OTP 20.

(HTTPoison.Error) :eaddrinuse
-----------------------------

<https://appsignal.com/neko/sites/59f22b2a6a21db47fddd50be/incidents/163>:

```
HTTP request error: {
  {
    {
      %HTTPoison.Error{id: nil, reason: :eaddrinuse},
      [
        {Neko.Shikimori.HTTPClient, :request!, 5, [file: 'lib/neko/shikimori/http_client.ex', line: 8]},
        {Neko.Shikimori.HTTPClient, :make_request!, 3, [file: 'lib/neko/shikimori/http_client.ex', line: 45]},
        {Neko.Shikimori.HTTPClient, :get_achievements!, 1, [file: 'lib/neko/shikimori/http_client.ex', line: 28]},
        {Neko.Achievement.Store, :achievements, 1, [file: 'lib/neko/achievement/store.ex', line: 46]},
        {Agent.Server, :handle_call, 3, [file: 'lib/agent/server.ex', line: 23]},
        {:gen_server, :try_handle_call, 4, [file: 'gen_server.erl', line: 636]},
        {:gen_server, :handle_msg, 6, [file: 'gen_server.erl', line: 665]},
        {:proc_lib, :init_p_do_apply, 3, [file: 'proc_lib.erl', line: 247]}
      ]
    },
    {
      GenServer,
      :call,
      [#PID<0.13805.26>, {:update, #Function<1.31387221/1 in Neko.Achievement.Store.reload/2>}, 110000]
    }
  },
  {
    GenServer,
    :call,
    [
      {:via, Registry, {:user_handler_registry, 48519}},
      {
        :process,
        %Neko.Request{action: "put", id: 37660338, score: 7, status: "completed", target_id: 32998, user_id: 48519}
      },
      120000
    ]
  }
}
```

**solution**

1. <https://github.com/edgurgel/httpoison#connection-pools>
2. <http://coderstocks.blogspot.co.at/2016/01/sqs-throughput-over-https-with-elixir.html>
3. <https://elixirforum.com/t/odd-slowdowns-with-concurrent-https-requests-http-client-concurrency/1221/12>

on startup hackney creates a default pool of connections which can be reused
globally in application (for requests to the same host) but doesn't use this
pool - by default all connections are created and closed dynamically.

to use the default pool, add it to `hackney` options:

```elixir
HTTPoison.get("httpbin.org/get", [], hackney: [pool: :default])
```

**NOTE**

there's a good chance that this error has nothing to do with creating too many
concurrent connections (and consequently using connection pool wouldn't help) -
Elixir supports much more concurrent connections than are currently created.

<https://elixirforum.com/t/odd-slowdowns-with-concurrent-https-requests-http-client-concurrency/1221/7>:

> You should be able to do a lot more than 100 concurrent connections
> (I was testing my server with 60k concurrent connections)...

***UPDATE (2018-03-05)***

error hasn't occurred in the last 11 days since the fix was deployed
(2018-02-22 01:48).

***UPDATE (2018-03-09)***

error occurred again under heavy load.

1. <https://github.com/edgurgel/httpoison#explicit-pool-creation>

even though Elixir might support lots of concurrent connection but here we
are talking about outbound connections - not inbound web server connections
=> it looks like this should be configured on HTTPoison (hackney) level.

<https://github.com/benoitc/hackney#use-the-default-pool>:

> `timeout` is the time we keep the connection alive in the pool,
> `max_connections` is the number of connections maintained in the pool.

so I've created a custom hackney pool for shikimori with these options:

- `timeout`: 150_000 ms (default) → 30 * 60_000 ms
- `max_connections`: 50 (default) → 150

default values for these options can be found in:

- project: _deps/hackney/src/hackney.app.src_
- release: _\<app_name>/releases/\<release_name>/\<app>.script_
  (say, _/home/apps/neko/releases/0.1.0/neko.script_)

***UPDATE (2018-03-10)***

most likely this is what causes the error:

shikimori is rebooted and there are lots of timeout errors in shikimori
(not only when it's down but right after it's started as well because of
high incoming traffic) - this means connections are taken from the pool
but not returned till they time out (in 90 seconds). at some point all
connections in the pool (50 connections by default) are open waiting for
shikimori to respond and new incoming request causes `eaddrinuse` error.

still it's a little bit unclear why this error happened when I didn't use
the pool - AFAIU there was no limit on max number of open connections then.

***UPDATE (2018-03-12)***

I've compared CPU usage of application with and without connection pool - it's
approximately the same (less than 2% in average, peak usage is about 6-7%, max
100 RPM).

***UPDATE (2018-03-19)***

the error is still present - it occurs repeatedly within a short period of time
(about 1 minute).

***UPDATE (2018-03-20)***

1. <http://erlang.org/pipermail/erlang-questions/2013-September/075275.html>
2. <https://github.com/benoitc/hackney/pull/136>

<https://stackoverflow.com/a/14388707/3632318> (brilliant answer):

> A TCP/UDP connection is identified by a tuple of five values:
>
> {\<protocol>, \<src addr>, \<src port>, \<dest addr>, \<dest port>}
>
> Any unique combination of these values identifies a connection. As a result,
> no two connections can have the same five values, otherwise the system would
> not be able to distinguish these connections any longer.
>
> The protocol of a socket is set when a socket is created with the socket()
> function. The source address and port are set with the bind() function.
> The destination address and port are set with the connect() function.

> If SO_REUSEADDR is enabled on a socket prior to binding it, the socket can
> be successfully bound unless there is a conflict with another socket bound
> to **exactly** the same combination of source address and port.
>
> ...SO_REUSEADDR has an effect on wildcard addresses, good to know.
>
> If SO_REUSEADDR is set for the socket you are trying to bind, another socket
> bound to the same address and port in state TIME_WAIT is simply ignored,
> after all its already "half dead", and your socket can bind to exactly the
> same address without any problem.

> Most people know that bind() may fail with the error EADDRINUSE, however,
> when you start playing around with address reuse, you may run into the strange
> situation that connect() fails with that error as well. How can this be?
>
> Well, with address reuse, you can bind two sockets of the same protocol to
> the same source address and port. That means three of those five values are
> already the same for these two sockets. If you now try to connect both of
> these sockets also to the same destination address and port, you would create
> two connected sockets, whose tuples are absolutely identical.

socket lifecycle:

- socket is bound
- socket is connected
- socket is closed

so the issue may be with `reuseaddr` hackney option - didn't find this option
in hackney sources though. at least it's said to be turned on by default when
using the pool [here](https://github.com/vhf/parareq/commit/50818b).

the root of my problem is that I connect to the same destination address
(shikimori URL) and there's a high chance of connection tuple collisions
which would result into `EADDRINUSE` error.

what can I do?

- use different destination ports when making requests
  (obviously it's not an option in most cases)
- disable address reuse if you don't make too many concurrent connections
  (or else you might run out of ports)

  then new sockets will be bound to new ports each time without reusing
  existing sockets in `TIME_WAIT` state.

  running out of ports doesn't seem to be a problem for me since I have 100
  RPM max now and requests are not long-running - sockets should be closed
  pretty quickly allowing their ports to be reused when new sockets are bound
  (binding socket is setting its source address and port).

***UPDATE (2018-04-08)***

the error is back after 16 days of silence - I have run out of ideas.

(HTTPoison.Error) :connect_timeout
----------------------------------

[appsignal](https://appsignal.com/neko/sites/59f22b2a6a21db47fddd50be/incidents/142):

```
HTTP request error: {
  {
    {
      %HTTPoison.Error{id: nil, reason: :connect_timeout},
      [
        {Neko.Shikimori.HTTPClient, :request!, 5, [file: 'lib/neko/shikimori/http_client.ex', line: 9]},
        {Neko.Shikimori.HTTPClient, :make_request!, 3, [file: 'lib/neko/shikimori/http_client.ex', line: 43]},
        {Neko.Shikimori.HTTPClient, :get_user_rates!, 1, [file: 'lib/neko/shikimori/http_client.ex', line: 16]},
        {Neko.UserRate.Store, :user_rates, 1, [file: 'lib/neko/user_rate/store.ex', line: 54]},
        {Agent.Server, :handle_call, 3, [file: 'lib/agent/server.ex', line: 23]},
        {:gen_server, :try_handle_call, 4, [file: 'gen_server.erl', line: 636]},
        {:gen_server, :handle_msg, 6, [file: 'gen_server.erl', line: 665]},
        {:proc_lib, :init_p_do_apply, 3, [file: 'proc_lib.erl', line: 247]}
      ]
    },
    {
      GenServer,
      :call,
      [#PID<0.23945.1>, {:update, #Function<3.20078263/1 in Neko.UserRate.Store.reload/2>}, 90000]
    }
  },
  {
    GenServer,
    :call,
    [
      {:via, Registry, {:user_handler_registry, 78123}},
      {
        :process,
        %Neko.Request{action: "put", id: 36135257, score: 0, status: "watching", target_id: 20583, user_id: 78123}
      },
      120000
    ]
  }
}
```

**solution**

1. <https://hexdocs.pm/httpoison/HTTPoison.html#request/5>

try to increase connect timeout (`timeout` request option in HTTPoison).

could not compile dependency :appsignal, "mix compile" failed
-------------------------------------------------------------

```
$ mix test
...
==> appsignal
could not compile dependency :appsignal, "mix compile" failed. You can
recompile this dependency with "mix deps.compile appsignal", update it
with "mix deps.update appsignal" or clean it with "mix deps.clean appsignal"
** (UndefinedFunctionError) function Appsignal.System.agent_platform/0 is undefined or private
    Appsignal.System.agent_platform()
    mix_helpers.exs:12: Mix.Appsignal.Helper.verify_system_architecture/0
    /Users/tap/dev/iceperkbilling/deps/appsignal/mix.exs:17: Mix.Tasks.Compile.Appsignal.run/1
```

**solution**

```sh
$ mix deps.clean appsignal
$ mix deps.get
$ mix test
```

system_limit
------------

[appsignal](https://appsignal.com/neko/sites/59f22b2a6a21db47fddd50be/incidents/287):

```
HTTP request error: {
  {
    :system_limit,
    [
      {
        :erlang,
        :spawn_link,
        [
          :proc_lib,
          :init_p,
          [
            #PID<0.2386.88>,
            [Neko.UserHandler.DynamicSupervisor, Neko.Supervisor, #PID<0.1005.0>],
            Task.Supervised,
            :reply,
            [#PID<0.2386.88>, :nomonitor, {:"neko@127.0.0.1", #PID<0.2386.88>}, {Neko.Achievement, :load, [88533]}]
          ]
        ],
        []
      },
      {:proc_lib, :spawn_link, 3, [file: 'proc_lib.erl', line: 100]},
      {Task, :async, 3, [file: 'lib/task.ex', line: 339]},
      {Neko.Request, :load_user_data, 1, [file: 'lib/neko/request.ex', line: 52]},
      {Neko.Request, :process, 1, [file: 'lib/neko/request.ex', line: 27]},
      {Neko.UserHandler, :handle_call, 3, [file: 'lib/neko/user_handler/user_handler.ex', line: 59]},
      {:gen_server, :try_handle_call, 4, [file: 'gen_server.erl', line: 636]},
      {:gen_server, :handle_msg, 6, [file: 'gen_server.erl', line: 665]}
    ]
  },
  {
    GenServer,
    :call,
    [
      {:via, Registry, {:user_handler_registry, 88533}},
      {
        :process,
        %Neko.Request{action: "reset", id: nil, score: nil, status: nil, target_id: nil, user_id: 88533}
      },
      120000
    ]
  }
}
```

**solution**

1. <http://erlang.org/documentation/doc-5.8.5/doc/efficiency_guide/advanced.html>
2. <http://erlang.org/documentation/doc-5.8.5//erts-5.8.5/doc/html/erl.html#max_processes>
3. <https://groups.google.com/forum/#!msg/elixir-lang-talk/No1Qq0huj_E/3Zsr6cCil_oJ>
4. <https://github.com/bitwalker/distillery/blob/master/docs/files/vm.args.md>

maximum number of concurrent processes (created via `spawn_link`) is reached
(32768 by default).

<http://erlang.org/documentation/doc-5.8.5/doc/efficiency_guide/advanced.html>:

> The maximum number of simultaneously alive Erlang processes is by default 32768.
> This limit can be raised up to at most 268435456 processes at startup
> (see documentation of the system flag +P in the erl(1) documentation).
> The maximum limit of 268435456 processes will at least on a 32-bit architecture
> be impossible to reach due to memory shortage.

create _rel/vm.args_:

```diff
  ## Enable kernel poll and a few async threads
  ##+K true
  ##+A 5

+ ## Set the maximum number of concurrent processes (default is 32768)
+ +P 65536
```

this file would replace _vm.args_ generated by distillery → instead
of creating it from scratch it's better to copy already generated file
_releases/\<release_name>/vm.args_ (it can be found locally either in
_\_build/prod/rel/neko/_ or inside release from _.deliver/release/_)
and make changes to it.

provide this file via `vm_args` option in _rel/config.exs_:

```elixir
+ # cookie is set in vm.args
  environment :prod do
+   set vm_args: "rel/vm.args"
    set include_erts: true
    set include_src: false
-   set cookie: :"my_cookie"
  end
```

Could not compile "src/gettext_po_parser.yrl" because the application "parsetools" could not be found
-----------------------------------------------------------------------------------------------------

this error occurred when compiling project on build host:

```
$ mix bootleg.build
...
** (Mix) Could not compile "src/gettext_po_parser.yrl" because the application
  "parsetools" could not be found. This may happen if your package manager
  broke Erlang into multiple packages and may be fixed by installing the
  missing "erlang-dev" and "erlang-parsetools" packages
```

**solution**

1. <https://stackoverflow.com/questions/36512519>

Erlang wasn't installed (I thought it would be installed as Elixir dependency).
