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

NOTE: the classification below is purely mine.

## usage

in all cases specific CM is usually fetched from environment
at compile time (using `Application.get_env/3`):

- in a single place (see `FCM` for details)
- in an arbitrary application module on demand

  ```elixir
  # client module
  defmodule MyApp.Foo do
    @api Application.get_env(:my_app, :api)
    def foo, do: @api.fetch()
  end
  ```

## behaviour module (BM)

- pure BM

  ```elixir
  defmodule MyApp.API do
    @callback fetch() :: any()
  end
  ```

- BM combined with CM

  BM can be embedded inside CM directly or as a nested module
  (in the latter case the boundary between the two is more explicit).

  this pattern is useful when there are only 2 CMs (real and mock CMs)
  and it's tiresome to create a separate CM for real implementation
  each time (or you cannot come up with a proper name for this module).

  embedding BM directly:

  ```elixir
  defmodule MyApp.API do
    @callback fetch() :: any()

    @behaviour __MODULE__
    def fetch, do: 123
  end
  ```

  nesting BM inside CM:

  ```elixir
  defmodule MyApp.API do
    defmodule Behaviour do
      @callback fetch() :: any()
    end

    @behaviour Behaviour
    def fetch, do: 123
  end
  ```

## callback module (CM)

- pure CM

  ```elixir
  defmodule MyApp.API.HTTPClient do
    @behaviour MyApp.API
    def fetch, do: 123
  end
  ````

- CM combined with BM

  same as vice versa.

- facade (or proxy) CM (FCM)

  unlike pure CM, FCM is not a fully functional CM since it implements
  callbacks by delegating to specific CM fetched from environment.

  FCM is often combined with BM.

  => FCM acts as a facade or proxy forwarding calls to dynamically
  fetched adapter so that clients are not even aware that they are
  dealing with switchable adapter under the hood (since they don't
  have to fetch it from environment by themselves like in all other
  cases) - FCM looks just like an ordinary module from outside:

  ```elixir
  # FCM
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

  also FCM gives an opportunity to do some housekeeping
  (say, logging) before or after delegating to actual CM.
