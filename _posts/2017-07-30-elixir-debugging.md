---
layout: post
title: Elixir - Debugging
date: 2017-07-30 15:49:05 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

using `IEx.pry` (= `binding.pry` in Ruby)
-----------------------------------------

1. <http://blog.plataformatec.com.br/2016/04/debugging-techniques-in-elixir-lang/>

- add `IEx.pry` breakpoint

  ```elixir
  defmodule Foo do
    def bar do
      require IEx; IEx.pry
    end
  end
  ```

- run your application inside IEx

  say, for Elixir app
  (see [Elixir - Application]({% post_url 2017-09-24-elixir-application %})):

  ```elixir
  $ \iex -S mix run --no-halt
  ```

- finish pry session by calling `respawn()`

### using `IEx.pry` in tests

1. [Elixir - Testing]({% post_url 2017-06-04-elixir-testing %})

### using `IEx.pry` in deps

- add `IEx.pry` breakpoint

  for example:

  ```sh
  $ vi deps/bootleg/lib/bootleg/ssh.ex
  ```

- compile dependency

  ```sh
  $ mix deps.compile bootleg
  ```

- run your application or relevant Mix task inside IEx

  ```sh
  $ \iex -S mix bootleg.build
  ```

debugging dependencies
----------------------

sometimes it might be necessary to debug or temporarily change external
dependencies stored in _deps/_ directory:

```sh
$ mvim deps/detergentex/lib/detergentex.ex
$ mix deps.compile detergentex
$ mix -S iex
```

information about endpoint
--------------------------

1. <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/5>

```sh
$ bin/billing remote_console
iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint.Server
iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint
```

memory consumption
------------------

```elixir
IO.puts("before foo: " <> inspect(:erlang.memory(:total)))
foo()
IO.puts("after foo: " <> inspect(:erlang.memory(:total)))
```

remote debugging
----------------

1. <http://blog.plataformatec.com.br/2016/05/tracing-and-observing-your-remote-node/>
2. <https://gist.github.com/pnc/9e957e17d4f9c6c81294>
3. <http://erlang.org/doc/apps/observer/observer_ug.html>

- `foo_host` is a name of remote host from _.ssh/config_
- `foo` is a name of remote Erlang node running on `foo_host`
  (most likely it matches application name)

### find out epmd and node ports on remote machine

```sh
$ ssh foo_host 'epmd -names'
epmd: up and running on port 4369 with data:
name foo at port 99999
```

here `4369` is epmd port and `99999` is a node port.

### forward ports

configure forwarding of corresponding local ports:

```sh
$ ssh -N -L 4369:localhost:4369 99999:localhost:99999 foo_host
```

here all connections are forwarded, say, from localhost:4369 to foo_host:4369
(whenever connection is made to locahost:4369, it's forwarded to foo_host:4369).

### start a hidden node running observer

```sh
\iex --name 'debug@localhost' --cookie 'bar' --hidden -e ':observer.start'
```

**IMPORTANT**

make sure magic cookie matches the one from _foo/var/vm.args_
(not the one from automatically generated `~/.erlang.cookie`)!
otherwise you won't be able to connect to `foo` next.

### connect to remote node

| observer menu: `Nodes` → `Connect Node` (type `foo@127.0.0.1`)

**IMPORTANT**

make sure node name matches the one from _foo/var/vm.args_ exactly
(`foo@localhost` != `foo@127.0.0.1`).

### (how to) debug process inside BEAM

1. <https://www.youtube.com/watch?v=pO4_Wlq8JeI&feature=youtu.be&t=853>

the higher number of reductions is, the more process loads CPU:

```
iex> Process.list
  |> Enum.map(& {Process.info(&1, :reductions), &1})
  |> Enum.sort(&>=/2)
  |> Enum.take(5)
  |> Enum.each(&IO.inspect)
iex> pid = pid(0,4225,0)
iex> Process.info(pid, :current_stacktrace)
```

debug that process with `dbg` (for some reason Sasa Juric doesn't recommend
it for production):

```
iex> :dbg.tracer()
iex> :dbg.p(pid, [:call])
iex> :dbg.tpl(:_, []); :timer.sleep(1000); :dbg.stop()
```

debugging deployment
--------------------

1. <https://github.com/edeliver/edeliver#help>

use `--debug` option to run in shell debug mode, say:

``` sh
$ mix edeliver start production --debug
```
