---
layout: post
title: Elixir - OTP
date: 2017-05-28 01:01:59 +0300
access: public
comments: true
categories: [elixir, otp, ets]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

NOTE: `Neko` is a sample application name in all examples below.

workers
-------

> <https://freecontent.manning.com/little-elixir-and-otp-implementing-a-supervisor/>
>
> What are worker processes? They are usually processes that have implemented
> the GenServer, GenFSM or GenEvent behaviors.

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

<https://elixirforum.com/t/can-a-genserver-state-be-too-big-and-general-application-architecture/469/3>:

GenServers (and processes in general) should be used for runtime organization
(running different tasks separately) - not code organization: required logic
must be implemented in dedicated modules which are used from inside GenServers.

Agent
-----

1. <https://hexdocs.pm/elixir/Agent.html>
2. <https://elixirforum.com/t/looking-for-clarity-around-using-agent/4750>

all agent calls except for `cast` are synchronous!

Task
----

1. <https://hexdocs.pm/elixir/Task.html>
2. <https://gist.github.com/moklett/d30fc2dbaf71f3b978da115f8a5f8387>

in general it's always better to start supervised tasks:

> <https://elixirforum.com/t/when-are-agent-and-task-supervisor-useful/896/4>
>
> A frequently overlooked but important role of supervisors is proper cleanup
> of processes. When a supervisor terminates, all of it’s descendants will be
> taken down as well.
>
> Therefore every process in your system should reside in the supervision
> tree, even if you don’t want to restart them. In such cases you can use
> the temporary restart strategy which is the default for Task.Supervisor.
>
> Except for some temporary experiments I wouldn’t advise using Task.start
> (or GenServer.start) in production as that may lead to dangling processes
> which may in turn cause some strange behaviour of the system.

tasks are linked to the caller:

- `Task.Supervisor.async`
- `Task.Supervisor.async_stream`

  it's possible to kill a task when it times out by setting `:on_timeout`
  option to `:kill_task` so that the caller process doesn't exit and other
  tasks are able to finish.

  `:timeout` option sets the maximum amount of time to wait for a single
  task to reply - not for the whole collection to be processed. that is
  why it's possible to run millions of tasks provided they are complete
  within the time specified in `:timeout` option:

  ```elixir
  iex> MyApp.TaskSupervisor
  |> Task.Supervisor.async_stream(
    [1, 2, 3, 4, 5],
    fn x -> IO.puts(x); :timer.sleep(1000) end,
    max_concurrency: 1,
    timeout: 1500
    )
  |> Enum.to_list()
  1
  2
  3
  4
  5
  [ok: :ok, ok: :ok, ok: :ok, ok: :ok, ok: :ok]
  ```

tasks are not linked to the caller:

- `Task.Supervisor.async_nolink`
- `Task.Supervisor.async_stream_nolink`
- `Task.Supervisor.start_child`

tasks can be awaited:

- `Task.Supervisor.async`
- `Task.Supervisor.async_nolink`

tasks are always awaited:

- `Task.Supervisor.async_stream`
- `Task.Supervisor.async_stream_nolink`

`Task.Supervisor.start_child`:

- tasks are linked to the supervisor but not to the caller
- crashed tasks can be restarted

  it's made possible by the fact that tasks are linked to the supervisor, not
  the caller. when using other functions (not `start_child`) task supervisor
  must have `:temporary` restart strategy while in case of `start_child` this
  strategy may be `:temporary` (the default), `:transient` or `:permanent`.

- task cannot be awaited

  when using other functions it's possible either to await on the task or task
  is always awaited (like in case of `Task.Supervisor.async_stream` - we get
  result by triggering enumeration on a stream) - this function should be used
  for side-effects only.

Supervisor
----------

1. <https://hexdocs.pm/elixir/Supervisor.html>

<https://elixirforum.com/t/are-supervisor-processes-genserver-processes/1838>:

> Supervisors should be extremely lightweight with low risk of having
> their own bugs because their job is to restart other processes.

<https://hexdocs.pm/elixir/Supervisor.html>:

> A supervisor may be started directly with a list of children
> via start_link/2 or you may define a module-based supervisor
> that implements the required callbacks.

when supervisor is to be put under a supervision tree, it must be defined
as a module-based supervisor - once it's added to the list of children of
top-level supervisor, the latter will use this supervisor's `child_spec/1`
function to retrieve its child spec and start it.

NOTE: top-level supervisor is started via `start_link/2` directly:

```elixir
defmodule Neko.Application do
  use Application

  def start(_type, _args) do
    children = [
      Neko.Anime.Store,
      # ...
    ]

    opts = [strategy: :rest_for_one, name: Neko.Supervisor]
    Supervisor.start_link(children, opts)
  end
end
```

### child specifications (child specs)

1. <https://hexdocs.pm/elixir/Supervisor.html#module-start_link-2-init-2-and-strategies>
2. <https://github.com/elixir-lang/elixir/blob/v1.5.2/lib/elixir/lib/supervisor.ex#L608>
3. <https://github.com/elixir-lang/elixir/blob/v1.6.1/lib/elixir/lib/supervisor.ex#L566>

since from Elixir 1.5 `use GenServer`, `use Agent` and `use Supervisor`
define `child_spec/1` function (default implementation of child spec):

```
iex> Neko.Achievement.Store.Registry.child_spec(:hello)
%{
  id: Neko.Achievement.Store.Registry,
  start: {Neko.Achievement.Store.Registry, :start_link, [:hello]}
}
```

and it's possible to specify module only as a supervisor child:

```elixir
children = [
  Neko.UserRate.Store
]
```

so default `child_spec/1` has arity 1 and you have to supply one argument -
when only module is specified as supervisor child, default argument `[]` is
passed. it's okay if this argument is discarded in `Neko.Foo.start_link/1`.
otherwise (say, when argument is a name used to register current GenServer)
it's necessary to pass along some meaningful value:

```elixir
children = [
  {Neko.UserRate.Store.Registry, Neko.UserRate.Store.Registry}
]
```

in general supervisor child can be specified now in 4 ways:

- child spec map itself
- tuple with module and start argument
  (`{Neko.Foo, arg}` → `Neko.Foo.child_spec(arg)` is called)
- module (`Neko.Foo` → `Neko.Foo.child_spec([])` is called)
- *[DEPRECATED]* child spec tuple
  (generated by `Supervisor.Spec` helpers)

supervisor children are started by calling their start function
(`start_link/1` by default) - both start function and its arity
(determined by the number of specified arguments) can be changed
by providing a custom child spec (with a custom `:start` key):

- override `child_spec/1` function inside `Neko.Foo` or
- customize existing child spec with `Supervisor.child_spec/2`

in general keep arity of 1 - it's confusing when `start_link` and
`init` functions have different arities (see Jose Valim's quote below).

also supervisor can supervise an arbitrary module (say, `Neko.Bar`)
if that module implements `child_spec/1` function.

**NOTE**

`Neko.Foo.start_link/1` has nothing to do with GenServer behaviour -
it's a custom function of your module (named `start_link` by convention)
which starts GenServer process (by calling `GenServer.start_link/3`,
`Agent.start_link/2`, `Supervisor.start_link/3` or whatever inside).

#### about deprecated Supervisor.Spec helpers (worker/3 and supervisor/3)

Jose Valim (<http://disq.us/p/1nbxadq>):

> One of the reasons why Elixir v1.5 introduced the new child specs was
> exactly to settle on `start_link/1` and `init/1`. The previous approach
> where `start_link` received a variadic number of arguments and `init`
> received only one was very confusing.
>
> The sore thumb is :simple_one_for_one supervisor but we are planning
> to revisit it for Elixir v1.6. I hope the old ways will be eventually
> forgotten and the experience will be streamlined. :)

`Supervisor.Spec` helpers generate child spec tuple which consists of
values from corresponding child spec map.

when using these helpers, it's possible to pass as many arguments to
start function of underlying module as you want - up to max allowed
function arity 255 (which can be confusing - see Jose Valim's comment
above).

```
iex> Supervisor.Spec.worker(Neko.Achievement.Store.Registry, [:hello, :world])
{Neko.Achievement.Store.Registry,
 {Neko.Achievement.Store.Registry, :start_link, [:hello, :world]}, :permanent,
 5000, :worker, [Neko.Achievement.Store.Registry]}
```

### about extra argument passed to supervisor child's start function

1. <https://blog.carbonfive.com/2018/01/30/comparing-dynamic-supervision-strategies-in-elixir-1-5-and-1-6/>

this only happens when using both:

- supervisor with `:simple_one_for_one` strategy
- a new way to fetch a child spec with `child_spec/1` function

  supervised child is usually specified as either tuple with module and
  start argument or just module - in both cases start argument must be
  passed (explicit argument in the 1st case and default one `[]` in the
  2nd case) since the arity of `child_spec/1` function is 1 (of course
  we can define our own `child_spec` function with any arity - but then
  we would have to fetch child spec manually with `Foo.child_spec(1, 2)`
  while `child_spec/1` function is called by supervisor automatically).

  then when starting a new child with `Supervisor.start_child/2` we might
  pass some dynamic arguments which are appended to the list of arguments
  in child spec:

  ```elixir
  %{
    id: Foo,
    start: {Foo, :start_link, [[], dynamic_arg_1, dynamic_arg_2]}
  }
  ```

  so now it turns out that `Foo.start_link` is called with 2+ arguments
  the 1st argument being totally useless - it's here only to satisfy the
  arity of default implementation of `child_spec/1`.

this problem is fixed by `DynamicSupervisor` in Elixir 1.6: it allows to
pass child spec right into `start_child/2` instead of `Supervisor.init/2`
and specify only those arguments for child's start function that we really
need.

ETS
---

<https://elixirforum.com/t/data-caching-agents-or-ets/1614/8> (Sasa Juric):

> Agent is a simple solution that could work for smaller loads and a few client
> processes. ETS table should usually perform better, and can support concurrent
> clients, i.e. you could have simultaneous multiple readers/writers - something
> not possible with Agent/GenServer. It is however very limited in terms of atomic
> operations, so it’s mostly suitable for simple k-v stuff, and some concurrent
> counters.
>
> Personally, if I know that there will be multiple clients of a key-value store,
> I just go for ETS immediately, because I believe this is what it was made for.
> That being said, some cases are in the grey area, so starting with a simple Agent
> is a somewhat simpler and more flexible solution. Assuming you encapsulate cache
> operations with some module, switching to ETS should be easy, because you’ll
> likely need to change the implementation in only one module (the cache wrapper).
>
> Finally ... think carefully whether you even need a cache. All other things
> being equal, cacheless is better than cacheful (because of less complexity),
> so if you can get away without it, it will be the simplest solution.

linking and monitoring
----------------------

### exit signals

1. <https://crypt.codemancers.com/posts/2016-01-24-understanding-exit-signals-in-erlang-slash-elixir>
2. <http://disq.us/p/1i0j8f3>
3. <https://groups.google.com/forum/#!topic/elixir-lang-talk/vxOtIXdqiWw>

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

  `:kill` exit signal always terminates receiving process even if it traps
  exits.

- other exit reasons (including `:shutdown`)

  other exit signals terminate receiving process unless the latter
  traps exits - these signals will be received as messages then.

![exit signal cheatsheet](https://crypt.codemancers.com/assets/images/elixir_processes/elixir_exit_signal_cheatsheet-f3204f86.png)

### getting notification about terminated process

monitor is identified by a reference returned by _Process.monitor/1_ -
this reference can used to pattern match on messages from mailbox of
monitoring process which have the following format:

```elixir
{:DOWN, ref, :process, object, reason}
```

<http://disq.us/p/1g92p7f>:

> Yes, if a process is only interested in getting a DOWN message when another
> process terminates, using a monitor will suffice - no need of creating a link
> and trapping exits. Apart from the fact that monitors only send messages which
> will arrive in the inbox of the monitoring process, another important thing
> to note is that monitors are uni directional.

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
