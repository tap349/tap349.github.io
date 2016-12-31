---
layout: post
title: Phoenix notes
date: 2016-11-12 16:21:40 +0300
access: public
categories: [phoenix, rails]
---

extracts from different books
(primarily from "Programming Phoenix" by Dave Thomas).

<!-- more -->

* TOC
{:toc}

### running app

- `iex` - doesn't load your app (equivalent of `irb`)
- `iex -S mix` - loads your app by executing _mix.exs_ script
  (equivalent of `rails console`)
- `iex -S mix phoenix.server` - loads your app and launches web server
  (`rails console` inside `rails server` - or vice versa)

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

### prefixes

NOTE: it's not possible to perform joins across prefixes -
data in different prefixes must be completely isolated.

prefixes can be configured on different levels:

global prefix ->
  schema prefix ->
    query/repository operation/struct prefix

#### global prefix

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

#### schema prefix

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

#### query prefix

```elixir
> query = Ecto.Queryable.to_query Rumbl.User
#Ecto.Query<from u in Rumbl.User>
> Rumbl.Repo.all %{query | prefix: "new_prefix"}
[]
```

#### repository operation prefix

```elixir
> Rumbl.Repo.all Rumbl.User, prefix: "new_prefix"
```

#### struct prefix

if it's nil it means global prefix is used
("public" or as configured in environment config).

```elixir
> [video] = Rumbl.Repo.all Rumbl.Video
[%Rumbl.Video{__meta__: #Ecto.Schema.Metadata<:loaded, "videos">, ...}
> new_prefix_video = Ecto.put_meta(video, prefix: "new_prefix")
%Rumbl.Video{__meta__: #Ecto.Schema.Metadata<:loaded, "new_prefix", "videos">, ...}
```

### get query SQL

<https://hexdocs.pm/ecto/Ecto.Adapters.SQL.html#to_sql/3>

```elixir
> import Ecto.Query
Ecto.Query
> query = from p in Rumbl.User
#Ecto.Query<from u in Rumbl.User>
> Ecto.Adapters.SQL.to_sql :all, Rumbl.Repo, query
{"SELECT u0.\"id\", u0.\"name\", u0.\"username\", u0.\"password_hash\", u0.\"inserted_at\", u0.\"updated_at\" FROM \"users\" AS u0", []}
```

### models vs changesets

<http://blog.tokafish.com/rails-to-phoenix-getting-started-with-ecto/>:

> Typically, you'd work with a changeset for making modifications to a model
> via the repo, and you'd work with the model when fetching the data for display.
