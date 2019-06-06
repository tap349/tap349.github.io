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

***UPDATE***

`mix format` adds parens for types when they are referenced using their
qualified names (by specifying the module where they are defined - say,
`String.t()` or `Agent.on_start()`). this doesn't apply to most built-in
types since they are referenced as is: `any`, `atom`, `pid`, etc.

=> stick to this convention for both built-in and user-defined types: use
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

parens
------

1. <https://hexdocs.pm/elixir/Code.html#format_string!/2-parens-and-no-parens-in-function-calls>

### anonymous functions

use parens for anonymous functions of arity 1+ (just like for named functions):

```elixir
fn -> IO.puts("hello") end
fn(_) -> IO.puts("hello") end
fn(foo) -> IO.puts("hello #{foo}") end
fn(foo, bar) -> IO.puts("hello #{foo} and #{bar}") end
```

***UPDATE***

`mix format` removes parens for anonymous functions of any arity:

```elixir
Enum.reduce(achievements, %{}, fn x, acc ->
  Map.put(acc, comparison_key(x), x)
end)
```

### macros

1. <https://hexdocs.pm/elixir/Kernel.SpecialForms.html#with/1>

> <https://hexdocs.pm/elixir/Kernel.SpecialForms.html#with/1>
>
> As with any other function or macro call in Elixir, explicit parens can
> also be used around the arguments before the do/end block.
>
> The choice between parens and no parens is a matter of preference.

> <https://hexdocs.pm/elixir/Code.html#format_string!/2-parens-and-no-parens-in-function-calls>
>
> By default, Elixir will add parens to all calls except for ... calls that
> have do/end blocks.

use parens:

- when macro is called like ordinary function
- when macro has no `do/end` block

don't use parens:

- to emphasize DSL nature of macro
- when macro has `do/end` block

singular vs. plural namespace names
-----------------------------------

> <https://softwareengineering.stackexchange.com/a/75929>
>
> Use the plural for packages with homogeneous contents and the singular for
> packages with heterogeneous contents.

I tend to use plural names when namespace is used to group related modules by
function (`operations`, `serializers`, `workers`, etc.) and singular names when
it's used to group modules by their domain (say, schema or context namespace).

protocols
---------

> <https://github.com/christopheradams/elixir_style_guide/issues/84#issuecomment-264245982>
>
> The rule I usually follow:
>
> 1. Define protocol in its own file.
> 2. For stdlib types or types from dependencies define the implementations
>    in the same file.
> 3. For custom types define the implementation in the same file that defines
>    the struct.
> 4. When both protocol and type are external, all bets are off. I never had
>    to do this, but if I did, I would probably go with file named like the
>    protocol, that hosts the implementations for external types - trying to
>    make it similar to the situation from 2.
