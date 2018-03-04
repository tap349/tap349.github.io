---
layout: post
title: Elixir - Style Guide
date: 2017-12-12 02:41:29 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/christopheradams/elixir_style_guide>

typespecs
---------

1. <https://hexdocs.pm/elixir/typespecs.html>

don't use parens for both built-in and user-defined types:

```elixir
defmodule MyApp.Rule do
  @typep rule_t :: struct

  @callback worker_pool_config() :: keyword
  @callback achievements([rule_t], pos_integer) :: [%MyApp.Achievement{}]
  @callback reload() :: any
end
```

**UPDATE**

`mix format` adds parens for types when they are referenced using their
qualified names (by specifying the module where they are defined - say,
`String.t()` or `Agent.on_start()`). this doesn't apply to most built-in
types since they are referenced as is: `any`, `atom`, `pid`, etc.

=\> stick to this convention for both built-in and user-defined types: use
parens only when type is referenced using its qualified name (it's allowed
to always use parens but omit them where possible to reduce visual noise):

```elixir
@type rule_t :: %Neko.Rules.SimpleRule{}
@type rules_t :: MapSet.t(rule_t)

@spec set(String.t(), rules_t) :: :ok
def set(name \\ __MODULE__, rules) do
  Agent.update(name, fn _ -> rules |> calc() end)
end
```

anonymous functions
-------------------

use parens for anonymous functions of arity 1+
(just like for named functions):

```elixir
fn -> IO.puts("hello") end
fn(_) -> IO.puts("hello") end
fn(foo) -> IO.puts("hello #{foo}") end
fn(foo, bar) -> IO.puts("hello #{foo} and #{bar}") end
```

**UPDATE**

`mix format` removes parens for anonymous functions of any arity:

```elixir
Enum.reduce(achievements, %{}, fn x, acc ->
  Map.put(acc, comparison_key(x), x)
end)
```
