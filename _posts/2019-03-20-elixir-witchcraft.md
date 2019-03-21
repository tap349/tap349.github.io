---
layout: post
title: Elixir - Witchcraft
date: 2019-03-20 23:00:24 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

style guide
-----------

using operator aliases (say, `>>>/2` instead of `chain/2`) has a few drawbacks:

- you've got to wrap captured named functions in parentheses

  ```elixir
  %Algae.Either.Right{right: 5}
  ~> (&foo()) # <--- it's required to wrap in parens here
  |> bar()
  ```

  otherwise unwrapped data is passed down the chain: say, if `foo/1` returns
  10, for some reason it's passed as is to `bar/1` while it must be wrapped in
  container (`%Algae.Either.Right{right: 10}`) - this is how `Functor.lift/2`
  is meant to work.

  NOTE: it's okay to use anonymous functions without parentheses:

  ```elixir
  %Algae.Either.Right{right: 5}
  ~> fn x -> x + 1 end # <--- no need to wrap in parens here
  |> bar()
  ```

- Credo complains that pipe chain doesn't start with a raw value
  (`Credo.Check.Refactor.PipeChainStart` check)

  ```elixir
  %Algae.Either.Right{right: 5}
  ~> (&foo())
  |> bar() # <--- Credo reports issue for this line
  ```

- Elixir formatter doesn't play well with some operators

  consider this code:

  ```elixir
  %Algae.Either.Right{right: 5}
  >>> (&foo())
  ~> bar()
  ```

  and this is how it's formatted:

  ```elixir
  %Algae.Either.Right{right: 5} >>>
    (&foo())
  ~> bar()
  ```

  maybe it's because Elixir has built-in `Bitwise.>>>/2` operator.

as a result I see no other way but to refrain from using operator aliases at
all except for few cases:

- some operators still can be used with anonymous functions (when they don't
  break formatting)
- all operators still can be used without pipe or when statement doesn't spread
  across multiple lines
