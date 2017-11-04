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

## `quote` vs `Macro.escape/2`

1. <https://elixirforum.com/t/understand-macro-escape/405/2>

- `quote` returns AST of passed in code
- `Macro.espace/2` returns AST of passed in value

that is `Macro.espace/2` first evaluates passed expression
and then returns AST of the result while `quote` returns
AST of passed expression as is.
