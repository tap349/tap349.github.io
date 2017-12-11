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
