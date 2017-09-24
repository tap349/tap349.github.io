---
layout: post
title: Elixir - Application
date: 2017-09-24 19:21:41 +0300
access: public
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

## evaluate or compile

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

### compile

- `$ elixirc foo.exs`

  compiles file and executes it.

- `iex> c "foo.exs"`

  compiles file and loads it into IEx.

## run

1. <https://stackoverflow.com/a/30688873/3632318>

### inside IEx

```sh
$ iex -S mix
```

NOTE: `run` is default Mix task - specify it explicitly
      if it's necessary to pass task options.

### without IEx

```sh
$ mix run --no-halt
```

## start/stop

### in IEx

```sh
iex> Application.start(:neko)
iex> Application.stop(:neko)
iex> Application.stop(:logger)
iex> Application.ensure_all_started(:neko)
```
