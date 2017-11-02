---
layout: post
title: Elixir - Behaviour
date: 2017-11-01 10:10:45 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://hexdocs.pm/elixir/behaviours.html>
2. <https://hexdocs.pm/elixir/Application.html>

## usage

in all cases specific callback module is usually fetched from
environment at compile time (using `Application.get_env/3`):

- in a single place (see facade callback module)
- in an arbitrary application module on demand

  ```elixir
  # client module
  defmodule MyApp.Foo do
    @api Application.get_env(:my_app, :api)
    def foo, do: @api.fetch()
  end
  ```

## behaviour module

- pure behaviour module

  ```elixir
  defmodule MyApp.API do
    @callback fetch() :: any()
  end
  ```

- behaviour module combined with callback module

  behaviour module can be embedded directly or as a nested module
  (in latter case the boundary between the two is more explicit).

  this pattern is useful when there are only 2 callback modules
  (real and mock callback modules) and it's tiresome to create
  a separate callback module for real implementation each time
  (or you cannot come up with a proper name for this module).

  embedding behaviour module directly:

  ```elixir
  defmodule MyApp.API do
    @callback fetch() :: any()

    @behaviour __MODULE__
    def fetch, do: 123
  end
  ```

  nesting behaviour module inside callback module:

  ```elixir
  defmodule MyApp.API do
    defmodule Behaviour do
      @callback fetch() :: any()
    end

    @behaviour Behaviour
    def fetch, do: 123
  end
  ```

## callback module

- pure callback module

  ```elixir
  defmodule MyApp.API.HTTPClient do
    @behaviour MyApp.API
    def fetch, do: 123
  end
  ````

- callback module combined with behaviour module

  same as vice versa.

- facade (or proxy) callback module (FCM)

  unlike pure callback module, FCM is not a fully functional
  callback module since it implements callbacks by delegating
  to specific callback module fetched from environment.

  FCM is often combined with behaviour module.

  => FCM acts as a facade or proxy forwarding calls to dynamically
  fetched adapter so that clients are not even aware that they are
  dealing with switchable adapter under the hood (since they don't
  have to fetch it from environment by themselves like in all other
  cases) - FCM looks just like an ordinary module from outside:

  ```elixir
  # facade callback module
  defmodule MyApp.API do
    defmodule Behaviour do
      @callback fetch() :: any()
    end

    @behaviour Behaviour
    @api Application.get_env(:my_app, :api)

    defdelegate fetch, to: @api
  end

  # client module
  defmodule MyApp.Foo do
    def foo, do: MyApp.API.fetch()
  end
  ```

  also FCM gives an opportunity to do some housekeeping (say, logging)
  before or after delegating to actual callback module.
