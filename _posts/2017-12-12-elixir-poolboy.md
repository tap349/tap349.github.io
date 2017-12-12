---
layout: post
title: Elixir - Poolboy
date: 2017-12-12 02:10:31 +0300
access: public
comments: true
categories: [elixir]
---

sample steps to setup Poolboy worker pool.

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/thestonefox/elixir_poolboy_example>
2. <https://elixirschool.com/en/lessons/libraries/poolboy>
3. <http://hashnuke.com/2013/10/03/managing-processes-with-poolboy-in-elixir.html>

- add worker pool configuration

  _config/config.exs_:

  ```elixir
  config :my_app, :rule_worker_pool,
    name: :rule_worker_pool,
    module: MyApp.Rule.Worker,
    size: 10,
    timeout: 10_000
  ```

- define single entry for fetching worker pool configuration

  _lib/my_app/rule.ex_:

  ```elixir
  def worker_pool_config do
    Application.get_env(:my_app, :rule_worker_pool)
  end
  ```

- create worker

  _lib/my_app/rule/worker.ex_:

  ```elixir
  defmodule MyApp.Rule.Worker do
    use GenServer

    def start_link(state \\ []) do
      GenServer.start_link(__MODULE__, state)
    end

    def achievements(pid, user_id) do
      GenServer.call(pid, {:achievements, user_id})
    end

    def init(_) do
      {:ok, MyApp.Rule.all()}
    end

    def handle_call({:achievements, user_id}, _from, rules) do
      # achievements are calculated for specified user using
      # provided rules and user-specific data fetched inside
      # MyApp.Rule.achievements/2
      achievements = MyApp.Rule.achievements(rules, user_id)
      {:reply, achievements, rules}
    end
  end
  ```

- call worker synchronously to process request

  _lib/my_app/achievement/calculator.ex_:

  ```elixir
  config = MyApp.Rule.worker_pool_config()
  :poolboy.transaction(
    config[:name],
    fn(pid) -> apply(config[:module], :achievements, [pid, user_id]) end,
    config[:timeout]
  )
  ```

- add Poolboy supervisor to supervision tree

  Poolboy supervisor is used to manage worker pool.

  _lib/my_app/application.ex_:

  ```elixir
  def start(_type, _args) do
    import Supervisor.Spec, warn: false

    # Define workers and child supervisors to be supervised
    children = [
      # ...
      rule_worker_pool_child(),
      # ...
    ]

    opts = [strategy: :rest_for_one, name: MyApp.Supervisor]
    Supervisor.start_link(children, opts)
  end

  defp rule_worker_pool_child do
    config = MyApp.Rule.worker_pool_config()
    :poolboy.child_spec(
      config[:name],
      [{:name, {:local, config[:name]}},
        {:worker_module, config[:module]},
        {:size, config[:size]},
        {:max_overflow, 2}
      ])
  end
  ```

- list all available Poolboy workers

  1. <https://github.com/devinus/poolboy/issues/91>

  ```elixir
  iex> GenServer.call(MyApp.Rule.worker_pool_config()[:name], :get_avail_workers)
  [#PID<0.367.0>, #PID<0.366.0>, #PID<0.365.0>, #PID<0.364.0>, #PID<0.363.0>,
   #PID<0.362.0>, #PID<0.361.0>, #PID<0.360.0>, #PID<0.359.0>, #PID<0.358.0>]
  ```
