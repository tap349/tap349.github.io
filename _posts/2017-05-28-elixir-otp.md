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

## Supervisor

- <https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html#the-application-callback>
- <https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>

> Supervisors should be extremely lightweight with low risk of having
> their own bugs because their job is to restart other processes.

2 ways to add application supervisor:

- in application callback module itself

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application
    alias Neko.Achievement.StoreRegistry

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

- in separate module

  _lib/neko/supervisor.ex_:

  ```elixir
  defmodule Neko.Supervisor do
    use Supervisor
    alias Neko.Achievement.StoreRegistry

    def start_link do
      Supervisor.start_link(__MODULE__, :ok)
    end

    def init(:ok) do
      children = [
        worker(StoreRegistry, [StoreRegistry])
      ]

      supervise(children, strategy: :one_for_one)
    end
  end
  ```

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application
    alias Neko.Achievement.StoreRegistry

    def start(_type, _args) do
      Neko.Supervisor.start_link
    end
  end
  ```

## Exit signals

- <http://crypt.codemancers.com/posts/2016-01-24-understanding-exit-signals-in-erlang-slash-elixir/>
- <https://groups.google.com/forum/#!topic/elixir-lang-talk/vxOtIXdqiWw>

Exit signal is a special type of messages.

```elixir
Process.exit(pid, <exit reason>)
```

classification of exit signals by their exit reasons:

- `:normal`

  `:normal` exit signal is ignored by receiving process unless it
  traps exits in which case the signal will be received as a message.

- `:kill`

  `:kill` exit signal always terminates receiving process even if it traps exits.

- other exit reasons (including `:shutdown`)

  such signals terminate receiving process unless it trap exits
  in which case these signals will be received as messages.

![exit signal cheatsheet](http://crypt.codemancers.com/assets/images/elixir_processes/elixir_exit_signal_cheatsheet-6f1371dea9066489fe5a287abc81d460c2c85785c32efbbb65a5837bb98d635f.png)
