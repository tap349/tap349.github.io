---
layout: post
title: Elixir notes
date: 2016-12-31 13:05:42 +0300
access: public
categories: [elixir]
---

Elixir notes.

<!-- more -->

- Erlang shell - Eshell (`erl`)
- Elixir shell - IEx (`iex`)
- UNIX shell - bash, etc.

### running `exs` files

- `iex> c "test.exs"`

  compiles file in memory (without creating beam file) and runs it

- `iex> import_file "test.exs"`

  evaluates file in context of IEx (it's like typing it in IEx line by line)

- `$ elixir "file.exs"`

  it's like running any other script in UNIX shell

### function capturing (& notation)

`&` - function capture operator
`&()` - capture syntax

- create anonymous function using provided expression

  - `add_one = &(&1 + 1)` (form with parentheses)
  - `add_one = & &1 + 1` (form without parentheses)
  - `to_tuple = & {&1, &2}`

  anonymous functions defined using & operator must have at least
  one argument (placeholders &1, &2, etc.):

  ```sh
  iex> a = &(1)
  ** (CompileError) iex:29: invalid args for &, expected an expression in the format of &Mod.fun/arity, &local/arity or a capture containing at least one argument as &1, got: {1}
  iex> a = &(&1)
  #Function<6.52032458/1 in :erl_eval.expr/5>
  iex> a.(1)
  1
  ```

- get anonymous function that calls existing named function

  - `puts = &IO.puts/1`

  Elixir Guide:

  > this notation can actually be used to retrieve a named function
  > as a function type

  Programming Elixir:

  > you can give it (& operator) the name and arity of an existing
  > function, and it will return an anonymous function that calls it

  to my mind it's like getting reference to existing named function.

  ```sh
  iex> puts = &IO.puts/1
  &IO.puts/1
  iex> puts.(123)
  123
  :ok
  ```

  it's useful when you want pass existing function as another function argument:

  `Enum.each [1, 2, 3, 4], &IO.inspect/1`
