---
layout: post
title: Elixir - Mix and IEx
date: 2017-05-21 12:08:46 +0300
access: public
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

## Mix

> Mix assumes that we are in the development environment unless we tell it
> otherwise with MIX_ENV=<another_environment> mix some_task.

### common tasks

- `mix compile`

  compiles the whole project including dependencies

- `mix deps.compile [<package>]`

  compiles project dependencies only (say, when you need to
  recompile dependencies after editing their source code manually)

- `mix deps.get` = `bundle install`

  gets all missing and out of date dependencies
  (updates existing ones up to the version specified in _mix.lock_)

- `mix deps.clean --unused --unlock` = `bundle install`

  removes dependencies which are no longer listed in _mix.exs_
  (`--unused`) and updates _mix.lock_ (`--unlock`)

- `mix deps.update --all` = `bundle update`
- `mix deps.update <package>` = `bundle update <gem>`

  updates dependency and writes updated version to _mix.lock_

see [Phoenix]({% post_url 2016-11-12-phoenix %}) for Phoenix-specific Mix tasks.

tasks can be batched:

```sh
$ mix do deps.get, compile
```

### gotchas

- Mix doesn't allow a task to be run twice (just like Rake)

  1. <https://stackoverflow.com/questions/36846041>

  that is why it's impossible to create Mix alias in _mix.exs_
  that runs the same task multiple times with different arguments:

  ```elixir
  defp aliases do
    [
      # works
      "ecto.reset": ["ecto.drop", "ecto.setup"],
      # doesn't work
      "deploy.prod": [
        "edeliver build release",
        "edeliver deploy release production"
      ]
    ]
  end
  ```

  though it's allowed to alternate between 2 tasks, say:

  ```elixir
  defp aliases do
    [
      # works
      "edeliver.all": [
        "edeliver build release",
        "deps.get",
        "edeliver deploy release production"
      ]
    ]
  end
  ```

### mix.exs

1. <http://blog.plataformatec.com.br/2016/07/understanding-deps-and-applications-in-your-mixfile/>

- application name

  ```elixir
  def project do
    [
      app: :neko,
      ...
    ]
  end
  ```

  - stored in _neko.app_ (created when project is compiled)
  - used to start/stop application manually in IEx

- application inference

  1. <http://elixir-lang.org/blog/2017/01/05/elixir-v1-4-0-released/#application-inference>

  deps should be no longer listed in applications list explicitly:

  ```elixir
  def application do
    [
      # this line is longer necessary - all dependencies
      # are added to `applications` by default in Elixir 1.4
      #applications: [:httpoison],
      # specify only extra applications from Erlang/Elixir
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

## IEx

1. <https://stackoverflow.com/documentation/elixir/1283/iex-console-tips-tricks>

- Erlang shell - Eshell (`erl`)
- Elixir shell - IEx (`iex`)
- UNIX shell - Bash, etc.

### start/stop application manually

```sh
iex> Application.start(:neko)
iex> Application.stop(:neko)
iex> Application.stop(:logger)
iex> Application.ensure_all_started(:neko)
```

### quit IEx

1. <http://blog.plataformatec.com.br/2016/03/how-to-quit-the-elixir-shell-iex/>

- `<C-c>a<CR>` (graceful)
- `<C-g>q<CR>` (graceful)
- `<C-c><C-c>` (not graceful)
- `<C-\>` (not graceful)

only graceful ways to quit IEx save shell history (when using `erlang-history`).

**UPDATE (2017-07-28)**

since Erlang/OTP 20 shell history is supported out of the box - if using it
instead of `erlang-history` patch all ways to quit IEx allow to save shell
history except for the last one (`<C-\>`).

### shell history

1. <http://nithinbekal.com/posts/elixir-shell-history/>:

```sh
$ git clone https://github.com/ferd/erlang-history.git
$ cd erlang-history
$ sudo make install
```

shell history since from now is stored in _~/.erlang-hist.nonode@nohost_
(it's a binary file - not plain text).

**UPDATE**: shell history is now stored in _~/.erlang-history/_ - you might
            need to remove _~/.erlang-hist.nonode@nohost_ for it to work.

**NOTE**: for shell history to be saved quit IEx gracefully -
          using either `<C-c>a<CR>` or `<C-g>q<CR>` commands.

`erlang-history` must be compiled for each new version of Erlang/OTP kernel!

- get current version of Kernel library:

  ```sh
  $ erl
  1> application:which_applications().
  [{stdlib,"ERTS  CXC 138 10","3.2"},
  {kernel,"ERTS  CXC 138 10","5.1.1"}]
  ```
  current version is 5.1.1.

- update `erlang-history` repo:

  ```sh
  $ cd erlang-history
  $ git up
  ```

- make and install:

  ```sh
  $ cd erlang-history
  $ sudo make install
  Password:
  erl -make
  Recompile: src/5.1.1/group_history
  Recompile: src/5.1.1/group
  ...
  ebin/5.1.1/group.beam
  ebin/5.1.1/group_history.beam
  2 file(s) copied
  ```

  make sure `2 file(s) are copied` - if no files are copied it means
  current version of Kernel library is not supported and you should
  probably update `erlang-history` repo.

**UPDATE (2017-07-28)**

1. <https://hexdocs.pm/iex/IEx.html#module-shell-history>:

> From Erlang/OTP 20, it is possible to get shell history by passing some flags
> that enable it in the VM.

1. <https://github.com/ferd/erlang-history>:

> Since Erlang/OTP-20rc2, Shell history is supported out of the box
> (although initially disabled by default) through a port of this library to
> the Erlang/OTP code base. Enable the shell in these versions by setting the
> shell_history kernel environment variable to enabled with export
> ERL_AFLAGS="-kernel shell_history enabled" added to your environment variables.

_~/.zshenv_:

```zsh
export ERL_AFLAGS="-kernel shell_history enabled"
```

using `erlang-history` is no longer required
(though I haven't found an easy way to uninstall it).

### evaluate vs. compile in memory vs. compile

1. <https://github.com/elixir-lang/elixir/issues/5073>

> `ex` files are meant to be compiled<br>
> `exs` files are used for scripting

each module is compiled into its own bytecode (`beam`) file:

- if file doesn't contain modules no bytecode files are generated
- it doesn't matter if module is defined in `exs` or `ex` file<br>
  (file extension doesn't matter at all - module can be defined in `rb` file)
- module `Example` is compiled into `Elixir.Example.beam` file

#### evaluate

- code typed in IEx is always evaluated - not compiled
- code is evaluated iff when typed in IEx directly
- never benchmark by typing code in IEx directly

#### compile in memory

in these cases bytecode modules are not written to disk - only loaded in memory:

- `$ iex foo.exs`

  compiles file in memory and loads it into IEx.

- `$ elixir foo.exs`

  compiles file in memory and executes it
  (it's like running any other script in UNIX shell).

- `iex> import_file "foo.exs"`

  > evaluates the contents of the file as if it were directly typed into IEx

  in spite of what is said above imported file is compiled in memory.

#### compile

- `$ elixirc foo.exs`

  compiles file and executes it.

- `iex> c "foo.exs"`

  compiles file and loads it into IEx.

### get result of last evaluated expression (same as `_` in irb)

1. <https://hexdocs.pm/iex/IEx.Helpers.html#v/1>

```sh
iex> 123
123
iex> v()
123
```

### cancel multiline command

1. <https://stackoverflow.com/questions/27591417>
2. <https://hexdocs.pm/iex/1.0.5/IEx.html>

```sh
iex> foo =
...> #iex:break
```

### cancel reverse search

1. <http://readline.kablamo.org/emacs.html>

`<D-u><D-u>`

the first `<D-u>` cancels reverse search but leaves found command if
any (cursor is located at the end of the line), the second `<D-u>` deletes
the line from current cursor position backwards to the start of the line.

### recompile current Mix application (same as `reload!` in rails console)

1. <http://stackoverflow.com/a/36494891/3632318>

```sh
iex> recompile()
```

also it's possible to recompile specific module:

```sh
iex> r(Foo.Bar)
```

### suppress long output (same as `;` in irb)

1. <http://stackoverflow.com/a/39208906/3632318>

add another expression at the end of the line after semicolon:

```sh
iex> Foo.bar(); 0
0
```
