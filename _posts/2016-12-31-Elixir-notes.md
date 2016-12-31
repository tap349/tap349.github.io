---
layout: post
title: Elixir notes
date: 2016-12-31 13:05:42 +0300
access: public
categories: [elixir]
---

Elixir notes.

<!-- more -->

### anonymous functions

anonymous functions defined with & notation must have at least one argument
(placeholders &1, &2, etc.):

```elixir
iex(1)> a = &(1)
** (CompileError) iex:29: invalid args for &, expected an expression in the format of &Mod.fun/arity, &local/arity or a capture containing at least one argument as &1, got: {1}
iex(2)> a = &(&1)
#Function<6.52032458/1 in :erl_eval.expr/5>
iex(3)> a.(1)
1
```
