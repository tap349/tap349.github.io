---
layout: post
title: Elixir - OTP
date: 2017-05-28 01:01:59 +0300
access: public
comments: true
categories: [elixir, otp]
---

<!-- more -->

* TOC
{:toc}
<hr>

## [GenServer](https://hexdocs.pm/elixir/GenServer.html)

- <https://medium.com/@adammokan/elixir-genserver-call-vs-cast-ba89fafd8847>
- <https://groups.google.com/forum/#!topic/elixir-lang-talk/DCTXyGV791w>
- <https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>

GenServers:

- GenServer itself (used for business logic)
- Agent (GenServer that only saves state)
- Supervisor (GenServer not meant to be used for business logic)

> The benefit of an Agent over a GenServer is in the nomenclature.

Task is not a GenServer but you can use GenServer as a Task.

<https://www.dailydrip.com/topics/elixir/drips/supervising-tasks-and-agents>:

> Tasks and Agents are both built on GenServer. Tasks are purely computation,
> and Agents are purely state management. For everything in between, there's
> GenServer.

## [Supervisor](https://hexdocs.pm/elixir/Supervisor.html)

<https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>:

> Supervisors should be extremely lightweight with low risk of having
> their own bugs because their job is to restart other processes.

2 ways to define supervisor:

- dynamic supervisor (defined in application callback module)

  1. <https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html#the-application-callback>

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application

    def start(_type, _args) do
      import Supervisor.Spec, warn: false

      children = [worker(Neko.Achievement.Store.Registry, [])]

      opts = [strategy: :one_for_one, name: Neko.Supervisor]
      Supervisor.start_link(children, opts)
    end
  end
  ```

- module-based supervisor (defined in a separate module)

  1. <https://hexdocs.pm/elixir/Supervisor.html#module-module-based-supervisors>

  _lib/neko/supervisor.ex_:

  ```elixir
  defmodule Neko.Supervisor do
    # automatically imports Supervisor.Spec
    use Supervisor

    def start_link do
      Supervisor.start_link(__MODULE__, :ok, name: Neko.Supervisor)
    end

    def init(:ok) do
      children = [worker(Neko.Achievement.Store.Registry, [])]

      # strategy is passed to Supervisor.Spec.supervise/2 -
      # not to Supervisor.start_link/2 like for dynamic supervisor
      supervise(children, strategy: :one_for_one)
    end
  end
  ```

  _lib/neko/application.ex_:

  ```elixir
  defmodule Neko.Application do
    use Application

    def start(_type, _args) do
      Neko.Supervisor.start_link()
    end
  end
  ```

### child specifications (child specs)

1. <https://hexdocs.pm/elixir/Supervisor.html#module-child-specification>
2. <https://hexdocs.pm/elixir/Supervisor.html#module-start_link-2-init-2-and-strategies>
3. <https://github.com/elixir-lang/elixir/blob/v1.5.2/lib/elixir/lib/supervisor.ex#L573>
4. <https://hexdocs.pm/elixir/GenServer.html#start_link/3>

sample child spec:

```elixir
iex> Neko.Achievement.Store.Registry.child_spec(:hello)
%{id: Neko.Achievement.Store.Registry, restart: :permanent, shutdown: 5000,
  start: {Neko.Achievement.Store.Registry, :start_link, [:hello]},
  type: :worker}
```

supervisor is passed a list of children when started
(with `Supervisor.start_link/2`), each child can be specified in 3 ways:

- module (`MyApp.Foo` = `{MyApp.Foo, []}`)
- tuple with module and start argument (`{MyApp.Foo, arg}`)
- child spec (`MyApp.Foo.child_spec(arg)`)

when module (1) or tuple (2) are provided, supervisor retrieves child spec (3)
from the given module (passing specified start argument in case of tuple) -
so using child spec must be the only true way of specifying supervisor child.

NOTE: you cannot pass more than 1 argument to underlying `start_link/2` when using `child_spec/1`
      (and consequently when using tuple too) - it's possible to pass
      list or tuple but still it'll be a single argument for `start_link/2`.

**IMPORTANT**

supervisor can supervise any module that implements `start_link/2`
while not all such modules might implement `child_spec/1` as well -
say, modules wrapping Agent (though you can always do it by yourself).

that is why it's better to always use `Supervisor.Spec.worker/3` or
`Supervisor.Spec.supervisor/3` helper functions that generate child
spec for specified module.

but, unlike child spec shown above, this generated child spec is a
tuple whose elements are the values from corresponding child spec map
(so it turns out it's kinda the 4th way to specify supervisor child).

NOTE: now when using these helper functions it's possible to pass as
      many arguments to underlying `start_link/2` as you want (up to
      maximum allowed function arity - 255).

```elixir
iex> Supervisor.Spec.worker(Neko.Achievement.Store.Registry, [:hello])
{Neko.Achievement.Store.Registry,
 {Neko.Achievement.Store.Registry, :start_link, [:hello]}, :permanent, 5000,
 :worker, [Neko.Achievement.Store.Registry]}
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
