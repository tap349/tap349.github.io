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

in all cases specific callback module is usually fetched
from environment (using `Application.get_env/3`) - either
in a single place (in some kind of a facade module) or in
an arbitrary application module on demand.

patterns to organize behaviour and callback modules (my own classification):

## pure behaviour module

```elixir
# behaviour module
defmodule MyApp.API do
  @callback fetch() :: any()
end

# callback module
defmodule MyApp.API.HTTPClient do
  @behaviour MyApp.API
  def fetch, do: 123
end

# client module
defmodule MyApp.Foo do
  @api Application.get_env(:my_app, :api)
  def foo, do: @api.fetch()
end
```

## combined behaviour module

behaviour module is combined with callback module:
callbacks are implemented in behaviour module itself.

this pattern is useful when there are only 2 callback modules
(real and mock callback modules) and it's tiresome to create
a separate callback module for real implementation each time
(or you cannot come up with a proper name for this module).

### flattened behaviour module

behaviour module is flattened with callback module
(callbacks are defined and implemented in the same module):

```elixir
# behaviour and callback module in one
defmodule MyApp.API do
  @callback fetch() :: any()
  @behaviour __MODULE__
  def fetch, do: 123
end

# client module
defmodule MyApp.Foo do
  @api Application.get_env(:my_app, :api)
  def foo, do: @api.fetch()
end
```

### nested behaviour module

behaviour module is nested inside callback module:

```elixir
# behaviour and callback module in one
defmodule MyApp.API do
  defmodule Behaviour do
    @callback fetch() :: any()
  end

  @behaviour Behaviour
  def fetch, do: 123
end

# client module
defmodule MyApp.Foo do
  @api Application.get_env(:my_app, :api)
  def foo, do: @api.fetch()
end
```

the same pattern as before but in this case the boundary
between behaviour and callback modules is more explicit.

### facade behaviour module

in this case behaviour module can be either flattened or nested
but the difference is that it implements callbacks by delegating
to specific callback module fetched from environment (that is it
just provides dummy implementation of specified behaviour - it's
not a fully functional callback module itself).

=> behaviour module acts as a facade or proxy forwarding calls to
dynamically fetched adapter so that clients are not even aware that
they are dealing with switchable adapter under the hood (since they
don't have to fetch it from environment by themselves like in all
other cases) - behaviour module looks just like an ordinary module
from outside:

```elixir
# behaviour and callback module in one
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

also this scheme gives an opportunity to do some housekeeping
(say, logging) before or after delegating to callback module.
