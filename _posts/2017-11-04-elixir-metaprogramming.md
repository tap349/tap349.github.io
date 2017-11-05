---
layout: post
title: Elixir - Metaprogramming
date: 2017-11-04 15:42:53 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

## unquoting expressions

1. <https://elixirforum.com/t/how-to-quote-regular-expression/6311>

[AST literals](https://elixir-lang.org/getting-started/meta/quote-and-unquote.html):

```elixir
:sum         #=> Atoms
1.0          #=> Numbers
[1, 2]       #=> Lists
"strings"    #=> Strings
{key, value} #=> Tuples with two elements
```

AST literals can be unquoted directly inside a macro.

all other expressions must be converted to quoted expressions
with `Macro.escape/2` before being unquoted inside a macro.

note that there'll be no error at compile time if you unquote
not AST literal inside a macro - error will occur when you'll
try to evaluate resulting quoted expression:

```sh
iex> quote do: "123" == unquote(%{a: 1})
{:==, [context: Elixir, import: Kernel], ["123", %{a: 1}]}
iex> Code.eval_quoted(quote do: "123" == unquote(%{a: 1}))
** (CompileError) nofile: invalid quoted expression: %{a: 1}
    (stdlib) lists.erl:1354: :lists.mapfoldl/3
    (stdlib) lists.erl:1355: :lists.mapfoldl/3
iex> Code.eval_quoted(quote do: "123" == unquote(Macro.escape(%{a: 1})))
{false, []}
```

## `quote` vs. `Macro.escape/2`

1. <https://elixirforum.com/t/understand-macro-escape/405/2>

- `quote` returns AST of passed in code
- `Macro.escape/2` returns AST of passed in value

that is `Macro.escape/2` first evaluates passed expression and returns
AST of the result while `quote` returns AST of passed expression as is.
