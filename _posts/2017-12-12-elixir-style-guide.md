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
