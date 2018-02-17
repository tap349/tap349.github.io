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

NOTE: `Neko` is a sample application name in all examples below.

GenServer
---------

1. <https://hexdocs.pm/elixir/GenServer.html>
2. <https://medium.com/@adammokan/elixir-genserver-call-vs-cast-ba89fafd8847>
3. <https://groups.google.com/forum/#!topic/elixir-lang-talk/DCTXyGV791w>
4. <https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>

GenServer - generic server process.

GenServers:

- `GenServer` itself (GenServer, used for business logic)
- `Agent` (agent - GenServer used for storing state only)
- `Supervisor` (supervisor - GenServer not used for business logic)

> The benefit of an Agent over a GenServer is in the nomenclature.

`Task` (task) is not a GenServer but you can use GenServer as a `Task`.

<https://www.dailydrip.com/topics/elixir/drips/supervising-tasks-and-agents>:

> Tasks and Agents are both built on GenServer. Tasks are purely computation,
> and Agents are purely state management. For everything in between, there's
> GenServer.

Agent
-----

1. <https://hexdocs.pm/elixir/Agent.html>
2. <https://elixirforum.com/t/looking-for-clarity-around-using-agent/4750>

all agent calls except for `cast` are synchronous!

Task
----

1. <https://hexdocs.pm/elixir/Task.html>
2. <https://gist.github.com/moklett/d30fc2dbaf71f3b978da115f8a5f8387>

- with linking caller to the task

  <https://hexdocs.pm/elixir/Task.html#module-async-and-await>:

  > async tasks link the caller and the spawned process

  ```elixir
  [Neko.Task1, Neko.Task2]
  |> Enum.map(&Task.async(&1, :run, [user_id]))
  |> Enum.map(&Task.await(&1, 10_000))
  ```

- without linking caller to the task

  <https://hexdocs.pm/elixir/Task.html#module-async-and-await>:

  > use Task.start/1 or consider starting the task under
  > a Task.Supervisor using async_nolink or start_child

  _lib/my_app/application.ex_:

  ```elixir
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    children = [
      # ...
      # default value of :restart option is :temporary
      # (required when Task.Supervisor.async_nolink/2 is used)
      supervisor(Task.Supervisor, [[name: Neko.TaskSupervisor]])
    ]

    opts = [strategy: :rest_for_one, name: Neko.Supervisor]
    Supervisor.start_link(children, opts)
  end
  ```

  ```elixir
  [Neko.Task1, Neko.Task2]
  |> Enum.map(fn(task) ->
    Neko.TaskSupervisor
    |> TaskSupervisor.async_nolink(task, :run, [user_id])
  end)
  |> Enum.map(&Task.yield/1)
  |> Enum.each(fn
    {:ok, _result} -> :ok
    {:exit, {error, _stack}} -> raise(error)
    nil -> raise("timeout running task")
  end)
  ```

  the task doesn't crash the caller iff `Task.yield/2` is used
  to capture results (task crashes when `Task.await/2` is used).

Supervisor
----------

1. <https://hexdocs.pm/elixir/Supervisor.html>

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

1. <https://hexdocs.pm/elixir/Supervisor.html#module-start_link-2-init-2-and-strategies>
2. <https://github.com/elixir-lang/elixir/blob/v1.5.2/lib/elixir/lib/supervisor.ex#L573>
3. <https://hexdocs.pm/elixir/GenServer.html#start_link/3>

sample child spec:

```elixir
iex> Neko.Achievement.Store.Registry.child_spec(:hello)
%{
  id: Neko.Achievement.Store.Registry,
  start: {Neko.Achievement.Store.Registry, :start_link, [:hello]}
}
```

supervisor is passed a list of children when started with
`Supervisor.start_link/2`, each child can be specified in 4 ways:

- child spec map (`Neko.Foo.child_spec(arg)`)
- module (`Neko.Foo` = `{Neko.Foo, []}`)
- tuple with module and start argument (`{Neko.Foo, arg}`)
- *[DEPRECATED]* child spec tuple (`Supervisor.Spec` helpers)

when module (1) or tuple (2) are provided, supervisor retrieves
child spec map (3) from the given module using its `child_spec/1`
(also passing specified start argument in case of tuple (2)).

only 1 start argument can be passed to `Neko.Foo.child_spec/1` =>
**supervised module must implement start function start_link/1 that
has arity 1** as well. this restriction is usually circumvented by
passing a list as the only start argument which might contain multiple
arguments as its elements.

NOTE: `Neko.Foo.start_link/1` has nothing to do with GenServer behaviour -
      it's a custom function of your module (named `start_link` by convention)
      which starts GenServer process (by calling `GenServer.start_link/3`,
      `Agent.start_link/2`, `Supervisor.start_link/3` or whatever inside).
      still this function must have arity 1 to fit into a supervision tree
      (see previous paragraph).

also supervisor can supervise an arbitrary module that is not GenServer
(say, `Neko.Bar`) if it's given that module's child spec:

- provided my module itself

  in this case `Neko.Bar` module must implement `child_spec/1` that
  returns its child spec map which instructs supervisor how to start
  that module.

  in theory provided child spec might instruct supervisor to start
  module using another start function (not necessarily `start_link/1`)
  but I guess this is highly not recommended.

- *[DEPRECATED]* generated by `Supervisor.Spec` helpers

  in this case `Neko.Bar` module must implement `start_link`
  (with any arity - all arguments supplied to `Supervisor.Spec`
  helper will be passed down to `Neko.Bar.start_link`).

#### Supervisor.Spec helpers (worker/3 and supervisor/3)

1. <https://github.com/elixir-lang/elixir/blob/v1.5.2/lib/elixir/lib/supervisor.ex#L608>

NOTE: `Supervisor.Spec` helpers are deprecated now.

Jose Valim (<http://disq.us/p/1nbxadq>):

> One of the reasons why Elixir v1.5 introduced the new child specs was
> exactly to settle on `start_link/1` and `init/1`. The previous approach
> where `start_link` received a variadic number of arguments and `init`
> received only one was very confusing.
>
> The sore thumb is :simple_one_for_one supervisor but we are planning
> to revisit it for Elixir v1.6. I hope the old ways will be eventually
> forgotten and the experience will be streamlined. :)

these helpers generate child spec tuple which consists of values from
corresponding child spec map.

also it's possible to pass as many arguments to start function of
underlying module as you want - up to max allowed function arity 255
(which can be confusing - see Jose Valim's comment above).

```elixir
iex> Supervisor.Spec.worker(Neko.Achievement.Store.Registry, [:hello, :world])
{Neko.Achievement.Store.Registry,
 {Neko.Achievement.Store.Registry, :start_link, [:hello, :world]}, :permanent,
 5000, :worker, [Neko.Achievement.Store.Registry]}
```

linking and monitoring
----------------------

### exit signals

1. <http://crypt.codemancers.com/posts/2016-01-24-understanding-exit-signals-in-erlang-slash-elixir/>
2. <https://groups.google.com/forum/#!topic/elixir-lang-talk/vxOtIXdqiWw>

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

  1. <https://elixir-lang.org/getting-started/mix-otp/genserver.html#monitors-or-links>

  monitoring process will be notified of crashes, exits, etc. of monitored
  process via `handle_info/2` callbacks.

  it's a preferable way if you just want to be informed when another process
  terminates.

- create a link and trap exits

  this method is usually used in supervisors - it's an overkill
  if you just want to be informed when another process terminates.

  moreover it's considered an antipattern in most cases
  (<http://www.erlang.se/doc/programming_rules.shtml#HDR22>):

  > As few processes as possible should trap exit signals.

  <https://www.reddit.com/r/elixir/comments/3dlwhu>:

  > A cornerstone of the Erlang philosophy is to separate error handling
  > and application logic in separate processes, workers and supervisors.
  > Trapping exits in your application code mixes these concerns in the
  > same process.
