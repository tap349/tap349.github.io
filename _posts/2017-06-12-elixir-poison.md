---
layout: post
title: Elixir - Poison
date: 2017-06-12 13:23:18 +0300
access: public
categories: [elixir, poison]
---

<!-- more -->

## `Poison.decode!` vs. `Poison.Parser.parse!`

<https://stackoverflow.com/a/41788994/3632318>

> They seem to produce the same results unless you're using the more advanced
> options of Poison.decode!. I guess using Poison.decode! should be alright,
> although the Readme suggests using Poison.Parser.parse! for simple JSON parsing.

## using structs

<https://elixir-lang.org/getting-started/protocols.html#implementing-any>

derive only `Protocol.Encoder` protocol implementation for encoding struct:

```elixir
defmodule Neko.Achievement do
  @derive [Protocol.Encoder]
  defstruct ~w(id name)a
end
```

it's not necessary to derive `Protocol.Decoder` for decoding into struct
(though it won't raise an error).
