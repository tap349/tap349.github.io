---
layout: post
title: Phoenix - Ecto
date: 2017-07-27 15:37:03 +0300
access: public
categories: [phoenix, ecto]
---

<!-- more -->

* TOC
{:toc}

NOTE: there is no _schema.rb_ file!

- <https://hexdocs.pm/phoenix/1.3.0-rc.3/ecto.html>
- <https://hexdocs.pm/ecto/Ecto.html>

## migrations

- `mix ecto.migrate` - runs all pending migrations
- `mix ecto.rollback` - rollbacks last applied migration

primitive column types that can be used in migrations:
[Types and casting](https://hexdocs.pm/ecto/Ecto.Schema.html#module-types-and-casting).

## schemas

<https://hexdocs.pm/ecto/Ecto.Schema.html>

## repository

<https://hexdocs.pm/phoenix/1.3.0-rc.3/ecto.html>

> Our repo (MyApp.Repo) has three main tasks:
>
> - to bring in all the common query functions from Ecto.Repo
> - to set the otp_app name equal to our application name
> - to initialize the options passed to the database adapter in init/2

## changesets

- <https://hexdocs.pm/phoenix/1.3.0-rc.3/ecto.html>
- <http://cultofmetatron.io/2017/04/22/thinking-in-ecto---schemas-and-changesets/>

```elixir
def changeset(%User{} = user, attrs) do
  user
  |> cast(attrs, [:uuid])        # params.require(:user).permit(:uuid) (strong parameters)
  |> validate_required([:uuid])  # validates :uuid, presence: true (AM validations)
end
```

`cast/3` also casts parameters to types defined in schema.

if you communicate with `Repo` directly by passing `User` struct instead
of changeset all validations defined in changeset are bypassed of course -
error will be raised only if underlying data store returns error.

## associations

<http://blog.plataformatec.com.br/2015/08/working-with-ecto-associations-and-embeds/>

prefer defining associations to specifying foreign key columns in
schema definitions (or else you won't be able to use them in queries):

```elixir
schema "cards" do
  # define association:
  belongs_to :user, MyApp.User
  # instead of specifying FK column:
  #field :user_id, :id
  timestamps()
end
```

<https://hexdocs.pm/ecto/Ecto.html#module-other-topics>

associations can be preloaded in:

- in schema definition itself (can' find the source - probably not true)
- in query:

  ```elixir
  Repo.all from p in Post, preload: [:comments]
  ```

- a posteriori (after fetching record or collection):

  ```elixir
  posts = Repo.all(Post) |> Repo.preload(:comments)
  ```

## models vs changesets

<http://blog.tokafish.com/rails-to-phoenix-getting-started-with-ecto/>:

> Typically, you'd work with a changeset for making modifications to a model
> via the repo, and you'd work with the model when fetching the data for display.

## Ecto.Multi

- <https://hexdocs.pm/ecto/Ecto.Multi.html>
- <http://blog.danielberkompas.com/2016/09/27/ecto-multi-services.html>

- replace `before` callbacks with functions in changesets
- replace `after` callbacks with operations in Ecto.Multi

use it when you would need callbacks in AR: Ecto.Multi allows to pack functions
that should be called after main action (like `create` or `update`) - all these
functions are named operations in Ecto.Multi parlance.

moreover using Ecto.Multi allows to stop execution if some operation fails and
returns `{:error, reason}` - this error can be returned either automatically
from `Repo` function or manually from functions passed to `Ecto.Multi.run`.

in the end Ecto.Multi struct is usually passed to `Repo.transaction/1` which
rollbacks transaction if any operation fails however calling `Repo.transaction/1`
in the end is not obligatory (if you don't want all operations to be run in
transaction - for example, they write to filesystem which cannot be rolled back).

it all resembles using monads to handle errors in general and using
`dry-monads` and `dry-matcher` gems in Ruby in particular
([monads in Ruby]({% post_url 2017-02-27-monads-in-ruby %})).

but unlike `dry-matcher` Ecto.Multi stores additional information in case of
failure - not only do we have error itself but also operation name and changes
accumulated in previous succeeded operations.

in this regard Ecto.Multi acts more like `dry-transaction` gem which allows to
handle errors arising from particular steps (= operations) with the difference
that `dry-transaction` holds operations (service objects that respond to `call`)
from DI container while multi packs operations (`Repo` or arbitrary functions).

also don't forget that there exist several monad libraries for Elixir
(e.g. [MonadEx](https://github.com/rob-brown/MonadEx)) which allow to use
this error handling mechanism in any module while Ecto.Multi is used when
you deal with persistence and need something to replace callbacks
(that is Ecto.Multi is alternative to our custom operations in Rails projects
which both persist data and run `after_*` callbacks manually).

## Programming Phoenix notes

### associations

associations are always loaded explicitly!

preparation:

```elixir
> alias Rumbl.Repo
> alias Rumbl.User
> import Ecto.Query
```

build and insert association:

```elixir
> attrs = %{title: 'title', description: 'desc'}
> video = Ecto.build_assoc(user, :videos, attrs)
> video = Repo.insert!(video)
```

alternatively build association explicitly:

```elixir
> video = %Video{user_id: user.id, title: 'title', description: 'desc'}
> video = Repo.insert!(video)
```

load association and store it in model struct:

```elixir
> user = Repo.preload(user, :videos)
> videos = user.videos
```

load association without storing it in model struct:

```elixir
> query = Ecto.assoc(user, :videos)
> videos = Repo.all(query)
```

## prefixes

NOTE: it's not possible to perform joins across prefixes -
data in different prefixes must be completely isolated.

prefixes can be configured on different levels:

global prefix ->
  schema prefix ->
    query/repository operation/struct prefix

### global prefix

_config/dev.exs_:

```elixir
config :rumbl, Rumbl.Repo,
  adapter: Ecto.Adapters.Postgres,
  username: "postgres",
  password: "postgres",
  database: "rumbl_dev",
  hostname: "localhost",
  pool_size: 10,
  after_connect: {Postgrex, :query!, ["SET search_path TO new_prefix", []]}
```

### schema prefix

NOTE: since Ecto 2.1

```elixir
defmodule Rumbl.User do
  # model/0 in web/web.ex uses Ecto.Schema:
  # use Ecto.Schema
  use Rumbl.Web, :model

  @schema_prefix "new_prefix"
  schema "users" do
    field :name, :string
    field :username, :string
    field :password, :string, virtual: true
    field :password_hash, :string
    has_many :videos, Rumbl.Video

    timestamps
  end
```

### query prefix

```elixir
> query = Ecto.Queryable.to_query Rumbl.User
#Ecto.Query<from u in Rumbl.User>
> Rumbl.Repo.all %{query | prefix: "new_prefix"}
[]
```

### repository operation prefix

```elixir
> Rumbl.Repo.all Rumbl.User, prefix: "new_prefix"
```

### struct prefix

if it's nil it means global prefix is used
("public" or as configured in environment config).

```elixir
> [video] = Rumbl.Repo.all Rumbl.Video
[%Rumbl.Video{__meta__: #Ecto.Schema.Metadata<:loaded, "videos">, ...}
> new_prefix_video = Ecto.put_meta(video, prefix: "new_prefix")
%Rumbl.Video{__meta__: #Ecto.Schema.Metadata<:loaded, "new_prefix", "videos">, ...}
```

## get query SQL

<https://hexdocs.pm/ecto/Ecto.Adapters.SQL.html#to_sql/3>

```elixir
> import Ecto.Query
Ecto.Query
> query = from p in Rumbl.User
#Ecto.Query<from u in Rumbl.User>
> Ecto.Adapters.SQL.to_sql :all, Rumbl.Repo, query
{"SELECT u0.\"id\", u0.\"name\", u0.\"username\", u0.\"password_hash\", u0.\"inserted_at\", u0.\"updated_at\" FROM \"users\" AS u0", []}
```
