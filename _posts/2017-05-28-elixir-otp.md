---
layout: post
title: Elixir - OTP
date: 2017-05-28 01:01:59 +0300
access: public
categories: [elixir, otp]
---

<!-- more -->

## GenServer

- <https://groups.google.com/forum/#!topic/elixir-lang-talk/DCTXyGV791w>
- <https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>

GenServers:

- GenServer itself (used for business logic)
- Agent (GenServer that only saves state)
- Supervisor (GenServer not meant to be used for business logic)

> The benefit of an Agent over a GenServer is in the nomenclature.

Task is not a GenServer but you can use GenServer as a Task.

<https://www.dailydrip.com/topics/elixir/drips/supervising-tasks-and-agents>:

> Tasks and Agents are both built on GenServer. Tasks are purely computation, and
> Agents are purely state management. For everything in between, there's GenServer.

## Supervisor

<https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>:

> Supervisors should be extremely lightweight with low risk of having
> their own bugs because their job is to restart other processes.

[2 ways](https://hexdocs.pm/elixir/Supervisor.html) to define supervisor:

- dynamic supervisor (defined in [application callback module](https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html#the-application-callback))

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application

    alias Neko.Achievement.Store.Registry, as: StoreRegistry

    def start(_type, _args) do
      import Supervisor.Spec, warn: false

      children = [
        worker(StoreRegistry, [StoreRegistry])
      ]

      opts = [strategy: :one_for_one, name: Neko.Supervisor]
      Supervisor.start_link(children, opts)
    end
  end
  ```

- module-based supervisor (defined in a separate module)

  _lib/neko/supervisor.ex_:

  ```elixir
  defmodule Neko.Supervisor do
    # automatically imports Supervisor.Spec
    use Supervisor

    alias Neko.Achievement.Store.Registry, as: StoreRegistry

    @name Neko.Supervisor

    def start_link do
      Supervisor.start_link(__MODULE__, :ok, name: @name)
    end

    def init(:ok) do
      children = [
        worker(StoreRegistry, [StoreRegistry])
      ]

      # NOTE: strategy is passed to Supervisor.Spec.supervise/2 -
      #       not to Supervisor.start_link/2 like for dynamic supervisor
      supervise(children, strategy: :one_for_one)
    end
  end
  ```

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application

    def start(_type, _args) do
      Neko.Supervisor.start_link
    end
  end
  ```

## linking and monitoring

### exit signals

- <http://crypt.codemancers.com/posts/2016-01-24-understanding-exit-signals-in-erlang-slash-elixir/>
- <https://groups.google.com/forum/#!topic/elixir-lang-talk/vxOtIXdqiWw>

exit signal is a special type of message.

exit signal is received by linked processes when a process terminates or when
exit signal (usually `:kill` exit signal) is sent to target process explicitly:

```elixir
Process.exit(pid, exit_reason)
```

classification of exit signals by their exit reasons:

- `:normal`

  `:normal` exit signal is ignored by receiving process unless the latter
  traps exits - this signal will be received as a message then.

- `:kill`

  `:kill` exit signal always terminates receiving process even if it traps exits.

- other exit reasons (including `:shutdown`)

  other exit signals terminate receiving process unless the latter
  traps exits - these signals will be received as messages then.

![exit signal cheatsheet](http://crypt.codemancers.com/assets/images/elixir_processes/elixir_exit_signal_cheatsheet-6f1371dea9066489fe5a287abc81d460c2c85785c32efbbb65a5837bb98d635f.png)

### getting notification about terminated process

<http://disq.us/p/1g92p7f>

- create a monitor

  <https://elixir-lang.org/getting-started/mix-otp/genserver.html#monitors-or-links>

  monitoring process will be notified of crashes, exits, etc. of monitored
  process via `handle_info/2` callbacks.

  it's a preferable way if you just want to be informed when another process
  terminates.

- create a link and trap exits

  this method is usually used in supervisors - it's an overkill if you just
  want to be informed when another process terminates.
