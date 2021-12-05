---
layout: post
title: Elixir - Enumerated Type
date: 2018-10-11 14:27:57 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <http://tech.honestbee.com/articles/elixir/2017-04/enums-in-elixir-ecto>
2. <http://nsomar.com/elixir-enumerated-types>

```elixir
# lib/lain/enum_type.ex

defmodule EnumType do
  defmacro __using__(values) do
    quote do
      def get(value) when value in unquote(values) do
        value
      end

      def get(value) do
        raise "Invalid enum value: #{value}"
      end
    end
  end
end
```

```elixir
# lib/lain/enums.ex

defmodule Lain.Enums do
  defmodule API.Jira do
    defmodule ProjectKey do
      use EnumType, ~w(
        CC
        AC
        DEV
        PROD
      )
    end
  end
end
```

usage:

```elixir
Lain.Enums.API.Jira.ProjectKey.get("DEV")
```
