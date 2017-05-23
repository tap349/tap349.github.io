---
layout: post
title: Elixir tips
date: 2017-05-21 12:08:46 +0300
access: public
categories: [elixir]
---

<!-- more -->

### mix tasks

- `mix deps.get` = `bundle install`

  installs new dependencies and updates existing ones up to the version
  specified in _mix.lock_

- `mix deps.clean --unused --unlock` = `bundle install`

  removes dependencies which are no longer mentioned in _mix.exs_
  (`--unused`) and updates _mix.lock_ (`--unlock`)

### mix.exs

- application inference

  <http://elixir-lang.org/blog/2017/01/05/elixir-v1-4-0-released/#application-inference>

  deps should be no longer listed in applications list explicitly:

  ```elixir
  def application do
    [
      # this line is longer necessary
      #applications: [:httpoison],
      extra_applications: [:logger],
      mod: {Neko.Application, []}
    ]
  end

  defp deps do
    [
      {:httpoison, "~> 0.11.1"}
    ]
  end
  ```

### IEx

#### quit IEx

<http://blog.plataformatec.com.br/2016/03/how-to-quit-the-elixir-shell-iex/>

- `<C-c>a<CR>` (graceful)
- `<C-g>q<CR>` (graceful)
- `<C-c><C-c>` (not graceful)
- `<C-\>` (not graceful)

only graceful ways to quit IEx allow to save shell history
(when using `erlang-history`).

#### get result of last evaluated expression (same as `_` in irb)

<https://hexdocs.pm/iex/IEx.Helpers.html#v/1>

```sh
iex> 123
123
iex> v()
123
```

#### cancel multiline command

- <https://stackoverflow.com/questions/27591417>
- <https://hexdocs.pm/iex/1.0.5/IEx.html>

```sh
iex> foo =
...> #iex:break
```

#### cancel reverse search

<http://readline.kablamo.org/emacs.html>

`<D-u><D-u>`

the first `<D-u>` cancels reverse search but leaves found command if
any (cursor is located at the end of the line), the second `<D-u>` deletes
the line from current cursor position backwards to the start of the line.

#### recompile current Mix application (same as `reload!` in rails console)

<http://stackoverflow.com/a/36494891/3632318>

```sh
iex> recompile()
```

also it's possible to recompile specific module:

```sh
iex> r Foo.Bar
```

#### debugging (same as `binding.pry` in Ruby)

<http://blog.plataformatec.com.br/2016/04/debugging-techniques-in-elixir-lang/>

- add `IEx.pry` breakpoint

  ```elixir
  require IEx

  defmodule Test do
    def foo do
      IEx.pry
    end
  end
  ```

- start new IEx session or recompile module with breakpoint
- finish pry session by calling `respawn`

#### suppress long output (same as `;` in irb)

<http://stackoverflow.com/a/39208906/3632318>

add another expression at the end of the line after semicolon:

```sh
iex> Foo.bar(); 0
0
```
