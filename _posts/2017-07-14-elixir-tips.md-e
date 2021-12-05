---
layout: post
title: Elixir - Tips
date: 2017-07-14 03:27:06 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://github.com/blackode/elixir-tips>

(how to) convert struct <-> map
-------------------------------

[struct to map](https://hexdocs.pm/elixir/Map.html#from_struct/1):

```elixir
Map.from_struct(%User{id: 1, name: "Alice"})
```

[map to struct](https://hexdocs.pm/elixir/Kernel.html#struct/2):

```elixir
# ignores keys that don't belong to struct
struct(User, %{id: 1, name: "Alice"})
# raises if key doesn't belong to struct
struct!(User, %{id: 1, name: "Alice", foo: 123})
# ignores keys that don't belong to struct
# (using ExConstructor package)
User.new(%{id: 1, name: "Alice", foo: 123})
```

(how to) update struct
----------------------

```elixir
user = %User{id: 1, name: "Alice"}

# raises if key doesn't belong to struct
user = %{user | foo: "Kate"}
# raises if key doesn't belong to struct
user = struct!(user, %{foo: "Kate"})
# ignores keys that don't belong to struct
user = struct(user, %{foo: "Kate"})
# adds keys that don't belong to struct
user = Map.put(user, :foo, "Kate")
# adds keys that don't belong to struct
user = Map.merge(user, %{foo: "Kate"})
```

after adding unknown keys to struct using `Map` module functions struct is no
longer recognized as original struct - now it's just a map with a `__struct__`
field.

(how to) remove newline inside or at the end of heredoc
-------------------------------------------------------

1. <https://elixirforum.com/t/line-break-at-the-end-of-a-multiline-string/12694/4>

```
iex> """
...> foo \
...> bar\
...> """
"foo bar"
```

(how to) get current environment at runtime
-------------------------------------------

> <https://stackoverflow.com/a/44747870>
>
> Mix.env doesn't work in production or other environments where you use
> compiled releases (built using Exrm/Distillery) or when Mix just isn't
> available.

=> store environment in application configuration since Mix is not available
in compiled application:

```elixir
# config/config.exs

config :my_app,
  env: Mix.env(),
  ecto_repos: [MyApp.Repo]
```

```elixir
defmodule MyApp.Foo do
  @env Application.get_env(:my_app, :env)

  def call do
    if @env == :prod do
      # production code
    else
      # non-production code
    end
  end
end
```

(how to) convert module name to string without `Elixir` prefix
--------------------------------------------------------------

```elixir
Macro.to_string(MyApp.Foo)
# => "MyApp.Foo"
```

```elixir
MyApp.Foo
|> Module.split()
|> Enum.join(".")
# => "MyApp.Foo"
```

(how to) measure function execution time
----------------------------------------

1. <https://stackoverflow.com/a/29674651/3632318>
2. <https://medium.com/elixirlabs/implement-a-basic-block-yield-with-elixir-d00f313831f7>

```elixir
defmodule Timer do
  defmacro time_ms(do: body) do
    quote do
      # or surround with calls to System.monotonic_time/1
      {time, result} = :timer.tc(fn -> unquote(body) end)
      {round(time / 1_000), result}
    end
  end
end
```

usage:

```elixir
require Timer

{time, result} =
  Timer.time_ms do
    Operation.call()
  end
```

don't use `bind_quoted` option of `Kernel.SpecialForms.quote/2` macro to pass
`body` binding to macro - execution time will be always 0 then (it looks like
execution time of unquoting `body` is measured - not execution time of `body`
itself though IDK for sure, just a wild guess):

```elixir
defmodule Timer do
  defmacro time_ms(do: body) do
    quote bind_quoted: [body: body] do
      # time is always 0 in this case
      # (the same when using System.monotonic_time/1)
      {time, result} = :timer.tc(fn -> body end)
      {round(time / 1_000), result}
    end
  end
end
```

don't use `require` when importing macros
-----------------------------------------

if macro is imported from another module, it's not necessary to `require` that
module:

```elixir
# when macro is not imported
require Foo
Foo.my_macro()

# when macro is imported
import Foo, only: [my_macro: 0]
my_macro()
```
