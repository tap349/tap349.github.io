---
layout: post
title: Elixir - Absinthe
date: 2019-02-01 12:02:00 +0300
access: public
comments: true
categories: [elixir, absinthe]
---

<!-- more -->

* TOC
{:toc}
<hr>

tips
----

### JSON scalar type

1. <https://hexdocs.pm/absinthe/custom-scalars.html>
2. <https://github.com/absinthe-graphql/absinthe/wiki/Scalar-Recipes#json-using-jason>
3. <https://elixirforum.com/t/absinthe-and-storing-jsonb/6741/2>
4. <https://github.com/absinthe-graphql/absinthe/blob/master/lib/absinthe/type/custom.ex>

```elixir
# lib/my_app_web/schema/custom_types.ex

defmodule MyAppWeb.Schema.CustomTypes do
  use Absinthe.Schema.Notation
  alias Absinthe.Blueprint.Input.{Null, String}

  scalar :json, name: "Json" do
    parse(&decode/1)
    serialize(&encode/1)
  end

  defp decode(%String{value: value}) do
    case Jason.decode(value) do
      {:ok, result} -> {:ok, result}
      _ -> :error
    end
  end

  defp decode(%Null{}), do: {:ok, nil}
  defp decode(_), do: :error

  # don't encode with Jason.encode!/2
  defp encode(value), do: value
end
```
