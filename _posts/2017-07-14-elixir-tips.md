---
layout: post
title: Elixir - Tips
date: 2017-07-14 03:27:06 +0300
access: public
categories: [elixir]
---

<!-- more -->

## ramblings about module naming in Elixir

as far as I can see folks prefer nouns to verbs when naming modules.

in this regard module acts like a namespace that groups related actions together.
say, in Ruby I might have several operations `User::Create`, `User::Update`, etc.
with a single `call` method (so called function objects) - in Elixir most likely
I would have a single `User` module with several functions `create`, `update`,
etc. inside.

moreover `User` module might define a struct and have functions working with
this struct inside the same module - of course struct must be passed to each
such function (modules are stateless).

if some functions inside `User` module can be grouped further it's possible to
create another module (= namespace) that is nested inside `User` (but usually
stored in a separate file in _user/_ directory) - say, `User.Authorization` module.
and so on and on...

that is why IMO Elixir modules tend to be bigger than Ruby classes.

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
