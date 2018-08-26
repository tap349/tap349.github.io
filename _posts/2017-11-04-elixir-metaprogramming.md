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
2. <https://dockyard.com/blog/2017/12/07/macro-madness-how-to-use-use-well>

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
------------------------

1. <https://elixirforum.com/t/understand-macro-escape/405/2>
2. <https://hexdocs.pm/elixir/Macro.html#escape/2-comparison-to-kernel-quote-2>

- `quote` returns AST of passed in code
- `Macro.escape/2` returns AST of passed in value

that is `Macro.escape/2` first evaluates passed expression and returns
AST of the result while `quote` returns AST of passed expression as is.

> <https://hexdocs.pm/elixir/Macro.html#escape/2-comparison-to-kernel-quote-2>
>
> Macro.escape/2 is used to escape values (either directly passed or variable
> bound), while Kernel.SpecialForms.quote/2 produces syntax trees for expressions.

bind_quoted option of quote/2
-----------------------------

> <https://hexdocs.pm/elixir/Kernel.SpecialForms.html>
>
> :bind_quoted - passes a binding to the macro. Whenever a binding is given,
> `unquote/1` is automatically disabled.

=> moreover unquoting is prohibited when using `bind_quoted` option:

```
iex> defmodule Foo do
...>   defmacro my_macro(name) do
...>     quote bind_quoted: [name: name] do
...>       IO.puts(unquote(name))
...>     end
...>   end
...> end
iex> require Foo
iex> Foo.my_macro("foo")
** (CompileError) iex:6: unquote called outside quote
    expanding macro: Foo.my_macro/1
    iex:6: (file)
```

> <https://dockyard.com/blog/2016/08/16/the-minumum-knowledge-you-need-to-start-metaprogramming-in-elixir>
>
> bind_quoted does two things:
>
> 1) prevent accidental reevaluation of bindings
> 2) defer the execution of `unquote` via `unquote: false`

all these rules (about `bind_quoted` and unquoting) don't apply to the
functions dynamically generated inside macros - you always can and have
to use `unquote` inside them:

```elixir
defmacro my_macro(name) do
  quote bind_quoted: [name: name] do
    IO.puts(name) # name is properly unquoted due to bind_quoted

    def foo do
      IO.puts(unquote(name)) # name must be always unquoted explicitly
    end
  end
end
```
