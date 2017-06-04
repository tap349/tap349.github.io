---
layout: post
title: Elixir - OTP
date: 2017-05-28 01:01:59 +0300
access: public
categories: [elixir, otp]
---

<!-- more -->

## GenServer vs. Agent

<https://groups.google.com/forum/#!topic/elixir-lang-talk/DCTXyGV791w>

> Agent is just a GenServer that only saves state.

> The benefit of an Agent over a GenServer is in the nomenclature.

Task is not a GenServer but you can use GenServer as a Task.

## Supervisor

- <https://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html#the-application-callback>

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
