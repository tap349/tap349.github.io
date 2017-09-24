---
layout: post
title: Elixir - IEx
date: 2017-05-21 12:08:46 +0300
access: public
categories: [elixir, iex]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://stackoverflow.com/documentation/elixir/1283/iex-console-tips-tricks>
2. <http://echobind.com/blog/2017-08-31-tips-and-tricks-for-iex/>

- Erlang shell - Eshell (`erl`)
- Elixir shell - IEx (`iex`)
- UNIX shell - Bash, etc.

## _.iex.exs_



## quit IEx

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

## shell history

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

## get result of last evaluated expression (same as `_` in pry)

1. <https://hexdocs.pm/iex/IEx.Helpers.html#v/1>

```sh
iex> 123
123
iex> v()
123
```

optional argument can be passed to return `n`th expression in current
IEx session: positive number indicates absolute position and negative
number indicates relative one (say, `v()` == `v(-1)`).

## cancel multiline command

1. <https://stackoverflow.com/questions/27591417>
2. <https://hexdocs.pm/iex/1.0.5/IEx.html>

```sh
iex> foo =
...> #iex:break
```

## cancel reverse search

1. <http://readline.kablamo.org/emacs.html>

`<D-u><D-u>`

the first `<D-u>` cancels reverse search but leaves found command if
any (cursor is located at the end of the line), the second `<D-u>` deletes
the line from current cursor position backwards to the start of the line.

## recompile current Mix application (same as `reload!` in pry)

1. <http://stackoverflow.com/a/36494891/3632318>

```sh
iex> recompile()
```

also it's possible to recompile specific module:

```sh
iex> r(Foo.Bar)
```

## suppress long output (same as `;` in pry)

1. <http://stackoverflow.com/a/39208906/3632318>

add another expression at the end of the line after semicolon:

```sh
iex> Foo.bar(); 0
0
```
