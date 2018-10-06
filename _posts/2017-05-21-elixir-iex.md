---
layout: post
title: Elixir - IEx
date: 2017-05-21 12:08:46 +0300
access: public
comments: true
categories: [elixir, iex]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>Erlang shell</dt>
  <dd>Eshell (erl)</dd>

  <dt>Elixir shell</dt>
  <dd>IEx (iex)</dd>

  <dt>UNIX shell</dt>
  <dd>Bash, etc.</dd>
</dl>

<hr>

1. <https://hexdocs.pm/iex/IEx.html>
2. <https://hexdocs.pm/iex/IEx.Helpers.html>
3. <https://stackoverflow.com/documentation/elixir/1283/iex-console-tips-tricks> (closed)
4. <http://echobind.com/blog/2017-08-31-tips-and-tricks-for-iex/>

.iex.exs
--------

IEx loads the 1st file it finds:

- _$PWD/.iex.exs_
- _$HOME/.iex.exs_

=> each project can have its own _.iex.exs_ with local aliases
that don't pollute global namespace.

=> IEx configs are not merged: if current project has its own
_.iex.exs_, user-wide config _~/.iex.exs_ is not read.

### aliases

contents of _.iex.exs_ is evaluated in shell's context so
it can be used to alias popular modules to cut down on typing:

```elixir
alias Neko.{Achievement, UserRate}
```

or else try to use [QuickAlias](https://github.com/thoughtbot/quick_alias)
package (though it wasn't working for me when I tried it).

quit IEx
--------

1. <http://blog.plataformatec.com.br/2016/03/how-to-quit-the-elixir-shell-iex/>

- `<C-c>a<CR>` (graceful)
- `<C-g>q<CR>` (graceful)
- `<C-c><C-c>` (not graceful)
- `<C-\>` (not graceful)

only graceful ways to quit IEx save shell history (when using `erlang-history`).

***UPDATE (2017-07-28)***

since Erlang/OTP 20 shell history is supported out of the box - if using it
instead of `erlang-history` patch all ways to quit IEx allow to save shell
history except for the last one (`<C-\>`).

shell history
-------------

<http://nithinbekal.com/posts/elixir-shell-history/>:

```sh
$ git clone https://github.com/ferd/erlang-history.git
$ cd erlang-history
$ sudo make install
```

shell history since from now is stored in _~/.erlang-hist.nonode@nohost_
(it's a binary file - not plain text).

UPDATE: shell history is now stored in _~/.erlang-history/_ - you might
need to remove _~/.erlang-hist.nonode@nohost_ for it to work.

for shell history to be saved quit IEx gracefully - using either `<C-c>a<CR>`
or `<C-g>q<CR>` commands.

`erlang-history` must be compiled for each new version of Erlang/OTP kernel:

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

***UPDATE (2017-07-28)***

<https://hexdocs.pm/iex/IEx.html#module-shell-history>:

> From Erlang/OTP 20, it is possible to get shell history by passing some flags
> that enable it in the VM.

<https://github.com/ferd/erlang-history>:

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

get result of last evaluated expression (same as `_` in pry)
------------------------------------------------------------

1. <https://hexdocs.pm/iex/IEx.Helpers.html#v/1>

```elixir
iex> 123
123
iex> v()
123
```

optional argument can be passed to return `n`th expression in current
IEx session: positive number indicates absolute position and negative
number indicates relative one (say, `v()` == `v(-1)`).

cancel multiline command
------------------------

1. <https://stackoverflow.com/questions/27591417>
2. <https://hexdocs.pm/iex/1.0.5/IEx.html>

```elixir
iex> foo =
...> #iex:break
```

cancel reverse search
---------------------

1. <http://readline.kablamo.org/emacs.html>

`<D-u><D-u>`

the first `<D-u>` cancels reverse search but leaves found command if
any (cursor is located at the end of the line), the second `<D-u>` deletes
the line from current cursor position backwards to the start of the line.

recompile current Mix application (same as `reload!` in pry)
------------------------------------------------------------

1. <http://stackoverflow.com/a/36494891/3632318>

```elixir
iex> recompile()
```

also it's possible to recompile specific module:

```elixir
iex> r(Foo.Bar)
```

suppress long output (same as `;` in pry)
-----------------------------------------

1. <http://stackoverflow.com/a/39208906/3632318>

add another expression at the end of the line after semicolon:

```elixir
iex> Foo.bar(); 0
0
```

don't truncate long lists or strings
------------------------------------

1. <https://stackoverflow.com/questions/29566248/elixir-io-inspect-to-not-trim-a-long-list>
2. <https://hexdocs.pm/elixir/Inspect.Opts.html>

for collections (defaults to 50 items):

```elixir
iex> IO.inspect(list, limit: :infinity)
```

for strings and charlists (defaults to 4096 bytes):

```elixir
iex> IO.inspect(list, printable_limit: :infinity)
```

force display a list of integers
--------------------------------

1. <https://github.com/elixir-lang/elixir/wiki/FAQ#4-why-is-my-list-of-integers-printed-as-a-string>
2. <https://www.theguild.nl/print-list-of-integers-as-integers-in-iex/>

see also [Elixir]({% post_url 2016-12-31-elixir %}) post (`character lists`
section) for ways to print charlist as a collection of codepoints.

### set `charlists` inspect option

1. <https://hexdocs.pm/elixir/Inspect.Opts.html>

NOTE: `Kernel.inspect/2` always returns a string.

```elixir
# in fact any value passed to `charlists` option except
# for `as_charlists` acts as `as_lists`:
iex> inspect [27, 35, 51], charlists: :as_lists
"[27, 35, 51]"
iex> inspect [27, 35, 51], charlists: :as_charlists
"'\\e#3'"
```

### add any non-printable character

`0` is usually added:

```elixir
iex> [27, 35, 51] ++ [0]
[27, 35, 51, 0]
```

### configure `inspect` option provided by IEx

1. <https://hexdocs.pm/iex/IEx.html#configure/1-inspect>

```elixir
iex> IEx.configure(inspect: [charlists: :as_lists])
iex> [27, 35, 51]
[27, 35, 51]
```

or else it can be set in IEx config file:

```elixir
# ~/.iex.exs

IEx.configure(inspect: [charlists: :as_lists])
```

this option will be respected by `IO.inspect/2` and by the shell when
printing results of expression evaluation - but not by `Kernel.inspect/2`.
