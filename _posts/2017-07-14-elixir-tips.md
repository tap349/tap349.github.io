---
layout: post
title: Elixir - Tips
date: 2017-07-14 03:27:06 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/blackode/elixir-tips>

ramblings about module naming in Elixir
---------------------------------------

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

***UPDATE***

the same principles apply to contexts (new feature of Phoenix 1.3) - they
group common functionality together (contexts are top-level domains of your
applicaiton). contexts can be further divided into subdomains (say, I have
such subdomains named after schemas - they contain operations, queries, etc.
that refer to (= bounded by) that specific schema).

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

(how to) remove newline at the end of heredoc
---------------------------------------------

1. <https://elixirforum.com/t/line-break-at-the-end-of-a-multiline-string/12694/4>

```
iex> """
...> foo\
...> """
"foo"
```
