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

1. <https://medium.com/@andreichernykh/elixir-a-bit-about-macros-behaviours-84fd3de1595d>

unquoting expressions
---------------------

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

```
iex> quote do: "123" == unquote(%{a: 1})
{:==, [context: Elixir, import: Kernel], ["123", %{a: 1}]}
iex> Code.eval_quoted(quote do: "123" == unquote(%{a: 1}))
** (CompileError) nofile: invalid quoted expression: %{a: 1}
    (stdlib) lists.erl:1354: :lists.mapfoldl/3
    (stdlib) lists.erl:1355: :lists.mapfoldl/3
iex> Code.eval_quoted(quote do: "123" == unquote(Macro.escape(%{a: 1})))
{false, []}
```

<https://elixir-lang.org/getting-started/meta/macros.html>:

> macro receives quoted expressions, injects them into the quote,
> and finally returns another quoted expression.

functions vs. macros
--------------------

<https://elixir-lang.org/getting-started/meta/macros.html>:

> arguments to a function call are evaluated before calling the function.
> macros do not evaluate their arguments - they receive the arguments as
> quoted expressions which are then transformed into other quoted expressions.

that is most likely macro arguments are quoted with `quote` before being
passed to macro itself.

quote vs. Macro.escape/2
----------------------------

1. <https://elixirforum.com/t/understand-macro-escape/405/2>

- `quote` returns AST of passed in code
- `Macro.escape/2` returns AST of passed in value

that is `Macro.escape/2` first evaluates passed expression and returns
AST of the result while `quote` returns AST of passed expression as is.
