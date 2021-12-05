---
layout: post
title: Elixir - Credo
date: 2018-01-07 21:35:18 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://github.com/rrrene/credo>

configuration
-------------

### generate Credo config

```sh
$ mix credo gen.config
```

generated config contains all Credo checks with default params.

usage
-----

### run Credo

suggestions + style guide on a per file basis:

```sh
$ mix credo list --strict
```

by default low priority issues are not shown - `--all-priorities` switch
(aliased as `--strict`) allows to include them in the output => there are
no special style guide checks enabled by this option.

explain specific issue (`explain` is a default command when
`filename:line_number:column` string is passed):

```sh
$ mix credo lib/neko/user_handler.ex:66:5
$ mix credo explain lib/neko/user_handler.ex:66:5
```

### disable check

```diff
  # .credo.exs

- {Credo.Check.Refactor.PipeChainStart, []},
+ {Credo.Check.Refactor.PipeChainStart, false},
```

### ignore Credo errors on a one-off basis

```elixir
# credo:disable-for-this-file
# credo:disable-for-this-file Credo.Check.Refactor.PipeChainStart

# credo:disable-for-next-line
Registry.fetch(user_id) |> Store.reload(user_id)
# credo:disable-for-next-line Credo.Check.Refactor.PipeChainStart
Registry.fetch(user_id) |> Store.reload(user_id)
# credo:disable-for-lines:2
MapSet.new(list_1)
|> MapSet.intersection(MapSet.new(list_2))
```

it's not required to re-enable checks just like for Rubocop.

Vim integration
---------------

### ale

_~/.vim/vimrc_:

```vim
let g:ale_linters = {
      \   'elixir': ['credo']
      \ }
```

ALE runs this command for `credo` linter:
`mix credo suggest --format=flycheck --read-from-stdin %s`.

the main point is that `--strict` option is missing so low priority issues
won't displayed by ALE.

this can be circumvented, if required, globally by switching to strict mode
(= show issues of all priorities):

```diff
  # .credo.exs

  %{
    configs: [
      %{
        # ...
-       strict: false,
+       strict: true,
        # ...
      }
    ]
  }
```

or by changing priority of check of interest:

```diff
  # .credo.exs

- {Credo.Check.Readability.MaxLineLength, priority: :low, max_length: 80},
+ {Credo.Check.Readability.MaxLineLength, priority: :normal, max_length: 80},
```

notes
-----

### check priorities

_.credo.exs_:

> Priority values are: `low, normal, high, higher`

each Credo check has its own default priority, priority of any check can be
customized with `priority` parameter in _.credo.exs_:

```elixir
{Credo.Check.Design.AliasUsage, priority: :low},
```

### issue categories

```sh
$ mix credo categories
```

- [C] consistency
- [D] design
- [R] readability
- [F] refactor
- [W] warning
