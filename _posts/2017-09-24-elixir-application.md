---
layout: post
title: Elixir - Application
date: 2017-09-24 19:21:41 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://hexdocs.pm/elixir/Application.html>

load paths
----------

1. <https://hexdocs.pm/mix/Mix.Tasks.Compile.Elixir.html>
2. <https://hexdocs.pm/elixir/Code.html#require_file/2>

Elixir compiles all source files found in `elixirc_paths`
(project configuration option in _mix.exs_).

`elixirc_paths` option can be set on per environment basis -
see [Elixir - Testing]({% post_url 2017-06-04-elixir-testing %}).

sometimes it's necessary to load specific file that is located
somewhere else (say, when running `exs` script file that depends
on another `exs` file) - use `Code.require_file/2` in this case:

```elixir
Code.require_file("benchmarks/benchmark.exs")
```

NOTE: when `relative_to` argument is not passed, all paths
      are considered to be relative to project root directory.

evaluate or compile
-------------------

1. <https://github.com/elixir-lang/elixir/issues/5073>

> `ex` files are meant to be compiled
>
> `exs` files are used for scripting

each module is compiled into its own bytecode (`beam`) file:

- if file doesn't contain modules no bytecode files are generated
- it doesn't matter if module is defined in `exs` or `ex` file
  (file extension doesn't matter at all - module can be defined in `rb` file)
- module `Example` is compiled into `Elixir.Example.beam` file

### evaluate

- code typed in IEx is always evaluated - not compiled
- code is evaluated iff when typed in IEx directly
- never benchmark by typing code in IEx directly

### compile in memory

in these cases bytecode modules are not written to disk - only loaded in memory:

- `$ iex foo.exs`

  compiles file in memory and loads it into IEx.

- `$ elixir foo.exs`

  compiles file in memory and executes it
  (it's like running any other script in UNIX shell).

- `iex> import_file "foo.exs"`

  > evaluates the contents of the file as if it were directly typed into IEx

  in spite of what is said above imported file is compiled in memory.

### compile to beam file

- `$ elixirc foo.exs`

  compiles file and executes it.

- `iex> c "foo.exs"`

  compiles file and loads it into IEx.

run
---

1. <https://stackoverflow.com/a/30688873/3632318>

NOTE: `iex` starts IEx without running your app (same as `irb`).

### Elixir application

- `mix run --no-halt` - run your Elixir app without IEx
- `iex -S mix` - start IEx and run your Elixir app inside
  (same as `rails console`)

NOTE: `run` is a default Mix task - specify it explicitly
      if it's necessary to pass any task options.

### Phoenix application

1. <https://hexdocs.pm/phoenix/up_and_running.html>

- `mix phx.server` - run your Phoenix app without IEx
- `iex -S mix phx.server` - start IEx and run your Phoenix app inside
  (looks like `rails console` inside `rails server`)

start/stop in IEx
-----------------

```sh
iex> Application.start(:neko)
iex> Application.stop(:neko)
iex> Application.stop(:logger)
iex> Application.ensure_all_started(:neko)
```
