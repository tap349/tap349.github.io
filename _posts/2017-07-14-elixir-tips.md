---
layout: post
title: Elixir - Tips
date: 2017-07-14 03:27:06 +0300
access: public
categories: [elixir]
---

<!-- more -->

## converting struct to map and vice versa

[struct to map](https://hexdocs.pm/elixir/Map.html#from_struct/1):

```elixir
Map.from_struct(%User{id: 1, name: "Alice"})
```

[map to struct](https://hexdocs.pm/elixir/master/Kernel.html#struct/2):

```elixir
# creates new struct
struct(User, %{id: 1, name: "Alice"})
# updates existing struct
struct(%User{id: 1, name: "Kate"}, %{name: "Alice"})
# raises if key doesn't belong to struct
struct!(User, %{id: 1, name: "Alice", foo: 123})
struct!(%User{id: 1, name: "Kate"}, %{foo: 123})
```

## updating struct

```elixir
user = %User{id: 1, name: "Alice"}

# raises if key doesn't belong to struct
user = %{user | name: "Kate"}
# raises if key doesn't belong to struct
user = struct!(user, %{name: "Kate"})
# ignores keys that don't belong to struct
user = struct(user, %{name: "Kate"})
# adds keys that don't belong to struct
user = Map.put(user, :name, "Kate")
# adds keys that don't belong to struct
user = Map.merge(user, %{name: "Kate"})
```

after adding unknown keys to struct using `Map` module functions struct is no
longer recognized as original struct - now it's just a map with a `__struct__`
field.
