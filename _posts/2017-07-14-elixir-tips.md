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

ramblings about module naming in Elixir and code organization
-------------------------------------------------------------

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
stored in a separate file in _user/_ directory) - `User.Authorization` module
for example. and so on and on...

that is why IMO Elixir modules tend to be bigger than Ruby classes.

***UPDATE***

the same principles apply to contexts (new feature of Phoenix 1.3) - they
group common functionality together (contexts are top-level domains of your
applicaiton). contexts can be further divided into subdomains (say, I have
such subdomains named after schemas - they contain operations, queries, etc.
that refer to (= bounded by) that specific schema).

### CQRS approach

1. <https://blog.lelonek.me/command-query-separation-in-elixir-ac742e60fc7d>

when using context most operations that constitute context public interface
and are used internally inside context are implemented inside context module
itself. complex operations can be extracted into their own modules (services,
operations) bounded by schemas as described above.

in CQRS approach it's recommended to separate read and write operations and
group them by related schema inside corresponding loaders and mutators. in
this case it's possible to expose only those operations via context public
interface (by importing these operations or by creating wrapper functions)
which are meant to be used outside current context (in other contexts or in
web part of Phoenix application). all other operations are considered to be
internal and should be used inside current context by calling functions from
required loaders and mutators directly.

implementing most operations in schema loaders and mutators and adding only
required ones to context public interface has the benefit of not polluting
the latter with operations which are not meant to be used outside current
context.

still it makes sense to extract complex operations (which might have private
helper functions) into their own modules (services, operations) within schema
namespace - just like when not using CQRS approach. these operations can use
different schema loaders and mutators and can be added to context interface.
for example, these operations can be placed inside `operations` namespace.

alternatively complex operations might be implemented as context functions
if they are just a sequence of calls to corresponding loaders and mutators.

### singular vs. plural namespace names

> <https://softwareengineering.stackexchange.com/a/75929>
>
> Use the plural for packages with homogeneous contents and the singular for
> packages with heterogeneous contents.

I tend to use plural names when namespace is used to group related modules by
function (`operations`, `serializers`, `workers`, etc.) and singular names when
it's used to group modules by their domain (say, schema or context namespace).

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

(how to) get current environment in production
----------------------------------------------

> <https://stackoverflow.com/a/44747870>
>
> Mix.env doesn't work in production or other environments where you use
> compiled releases (built using Exrm/Distillery) or when Mix just isn't
> available.

=> store environment in config since Mix is not available in production:

```elixir
# config/config.exs

# General application configuration
config :lain,
  env: Mix.env(),
  ecto_repos: [Lain.Repo]
```

```elixir
defmodule Lain.Foo do
  @env Application.get_env(:lain, :env)
end
```
