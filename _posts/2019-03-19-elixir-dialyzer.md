---
layout: post
title: Elixir - Dialyzer
date: 2019-03-19 00:00:31 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

installation
------------

1. <https://github.com/jeremyjh/dialyxir>

notes
-----

### PLT files

1. <https://github.com/jeremyjh/dialyxir#plt>
2. <https://hexdocs.pm/mix/1.1.1/Mix.Utils.html#mix_home/0>

when `MIX_HOME` environment variable is not set Mix home defaults to _~/.mix/_
but when using asdf it's _~/.asdf/installs/elixir/1.8.1/.mix/_ - this is where
core Erlang and Elixir PLT files are built.

when `mix dialyzer` task is run for the first time:

- core Erlang and Elixir PLT files are built
- project environment specific PLT file is copied from core Elixir PLT file and
  then augmented with project modules

PLT files are rebuilt for each new version of Erlang or Elixir.

configuration
-------------

### Slime templates

for some reason Dialyzer complains about Slime templates (probably it doesn't
recognize them) so I've opted just to ignore them:

```diff
  # mix.exs

  def project do
    [
      # ...
-     deps: deps()
+     deps: deps(),
+     dialyzer: [
+       ignore_warnings: ".dialyzer_ignore.exs"
+     ]
    ]
  end
```

```elixir
# .dialyzer_ignore.exs

[
  ~r/.*\.html\.slime.*/
]
```

### Phoenix support

1. <https://github.com/jeremyjh/dialyxir/wiki/Phoenix-Dialyxir-Quickstart>

```diff
  # mix.exs

  def project do
    [
      # ...
      deps: deps(),
      dialyzer: [
+       plt_add_deps: :transitive,
        ignore_warnings: ".dialyzer_ignore.exs"
      ]
    ]
  end
```

### integration with ALE (Vim plugin)

adding `dialyxir` linter to the list of Elixir linters has no effect -
linter is run when file is saved but no issues are shown. I don't want
to investigate it any further because running Dialyzer on each save is
quite costly IMO.
