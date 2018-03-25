---
layout: post
title: Phoenix - Ecto
date: 2017-07-27 15:37:03 +0300
access: public
comments: true
categories: [phoenix, ecto]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://hexdocs.pm/phoenix/ecto.html>
2. <https://hexdocs.pm/ecto/getting-started.html>
3. <https://hexdocs.pm/ecto/Ecto.html>

migrations
----------

1. <https://devhints.io/phoenix-migrations>

create migration:

```sh
$ mix ecto.gen.migration add_payment_service_to_transfers
```

run all pending migrations:

```sh
$ mix ecto.migrate
```

rollback last applied migration:

```sh
$ mix ecto.rollback
```

NOTE: there is nothing like _schema.rb_ file in Phoenix project -
      database schema is not dumped to file after running migrations.

primitive column types that can be used in migrations:
[Types and casting](https://hexdocs.pm/ecto/Ecto.Schema.html#module-types-and-casting).

repositories
------------

<https://hexdocs.pm/phoenix/ecto.html>:

> Our repo (MyApp.Repo) has three main tasks:
>
> - to bring in all the common query functions from Ecto.Repo
> - to set the otp_app name equal to our application name
> - to initialize the options passed to the database adapter in init/2

schemas
-------

1. <https://hexdocs.pm/ecto/Ecto.Schema.html>

changesets
----------

1. <https://hexdocs.pm/phoenix/ecto.html>
2. <http://cultofmetatron.io/2017/04/22/thinking-in-ecto---schemas-and-changesets/>

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

associations
------------

### associations vs. FK columns

1. <http://blog.plataformatec.com.br/2015/08/working-with-ecto-associations-and-embeds/>

prefer defining associations to specifying FK columns in schema
definitions (or else you won't be able to use them in queries):

```elixir
schema "cards" do
  # define association:
  belongs_to :user, MyApp.User
  # instead of specifying FK column:
  #field :user_id, :id
  timestamps()
end
```

### preloading associations

1. <https://hexdocs.pm/ecto/Ecto.html#module-other-topics>

assocations are never preloaded by default - load them explicitly!

associations can be preloaded in:

- in query when loading parent struct:

  ```elixir
  Repo.all from p in Post, preload: [:comments]
  ```

- a posteriori after loading parent struct:

  ```elixir
  posts = Repo.all(Post) |> Repo.preload(:comments)
  ```

associations can be loaded in:

- into parent struct:

  ```elixir
  post = Repo.preload(post, :comments)
  ```

- into its own struct:

  ```elixir
  query = Ecto.assoc(post, :comments)
  comments = Repo.all(query)
  ```

### building associations

#### `put_assoc` vs. `cast_assoc`

1. <https://medium.com/coryodaniel/til-elixir-ecto-put-assoc-vs-cast-assoc-7c80f35f6e6>
2. <https://hexdocs.pm/ecto/Ecto.Changeset.html#cast_assoc/3>

- when using `put_assoc`:

  association that you `put` usually already exists (that is it's not meant to
  be persisted as a result of current operation) and is supplied as a struct or
  changeset (say, `user` when creating a `comment`).

- when using `cast_assoc`:

  you specify the name of association that is expected to be created alongside
  its parent (this name is also used to retrieve association params from supplied
  `params`) - like when using `accepts_nested_attributes_for` in Rails.

though in both cases corresponding association must be preloaded so that
Ecto knows what to do in case it already exists (of course it only makes
sense when parent has `id` field - but it doesn't raise error otherwise).

#### `build_assoc` vs. building association explicitly

1. <https://elixirforum.com/t/why-do-we-have-ecto-build-assoc/4152>

```elixir
# build association using parent struct and association name
user = Repo.get_by(User, name: "John")
comment = Ecto.build_assoc(user, :comments, body: "foo")
```

=

```elixir
# build association explicitly by setting FK column value
user = Repo.get_by(User, name: "John")
comment = %Comment{user_id: user.id, body: "foo"}
```

> The goal of build_assoc is to allow you to work on
> the association names instead of the key names.

#### usage examples of `put_assoc`, `build_assoc` and `cast_assoc`

it's not recommended to use these functions inside schemas since they might
require you to make calls to `Repo` (`build_assoc` or `put_assoc`) or use
changesets of other schemas (`cast_assoc`) while it's preferable to isolate
both calls to `Repo` and usage of different changesets in contexts.

though in `What's new in Ecto 2.1` ebook (chapter `Many to many and upserts`)
there is an example of schema in which both `put_assoc` and `Repo` are used.

- `put_assoc`

  <https://elixirforum.com/t/using-put-assoc-in-changeset-for-many-to-many/3848/2>:

  > However, that put_assoc is about a belongs_to relationship and that expects
  > either nil or a struct.

  build new comment using its `user` association and existing user:

  ```elixir
  # in controller or context

  user = Repo.get_by(User, name: "John")

  comment
  |> Repo.preload(:user)
  |> Comment.changeset(params)
  |> put_assoc(:user, user)
  |> Repo.insert!

  # in schema

  def changeset(%Comment{} = comment, params) do
    comment
    |> cast(params, [:body])
    |> validate_required([:body])
  end
  ```

- `build_assoc`

  build new comment using existing user and his `comments` association:

  ```elixir
  # in controller or context

  user = Repo.get_by(User, name: "John")

  user
  |> build_assoc(:comments)
  |> Comment.changeset(params)
  |> Repo.insert!

  # in schema

  def changeset(%Comment{} = comment, params) do
    comment
    |> cast(params, [:body])
    |> validate_required([:body])
  end
  ```

- `cast_assoc`

  build new user with new comments to save them all later in one go:

  ```elixir
  # in controller or context

  user = %User{}

  user
  |> Repo.preload(:comments)
  |> User.changeset(params)
  |> cast_assoc(:comments, with: &Comment.insert_changeset/2)
  |> Repo.insert!

  # in schema

  # &Comment.changeset/2 is used by default
  # to create changesets for comments
  def changeset(%User{} = user, params) do
    user
    |> cast(params, [:name])
    |> validate_required([:name])
  end
  ```

models vs. changesets
---------------------

<http://blog.tokafish.com/rails-to-phoenix-getting-started-with-ecto/>:

> Typically, you'd work with a changeset for making modifications to a model
> via the repo, and you'd work with the model when fetching the data for display.

**UPDATE**:

models have been deprecated in Phoenix 1.3 - they are called just schemas now.

Ecto.Multi
----------

1. <https://hexdocs.pm/ecto/Ecto.Multi.html>
2. <http://blog.danielberkompas.com/2016/09/27/ecto-multi-services.html>
3. <https://medium.com/@feymartynov/an-example-of-using-ecto-multi-5f7fc8cf3cc1>

- replace `before` callbacks with functions in changesets
- replace `after` callbacks with operations in `Ecto.Multi`

use it when you would need callbacks in AR: `Ecto.Multi` allows to pack functions
that should be called after main action (like `create` or `update`) - all these
functions are named operations in `Ecto.Multi` parlance.

moreover using `Ecto.Multi` allows to stop execution if some operation fails and
returns `{:error, reason}` - this error can be returned either automatically
from `Repo` function or manually from functions passed to `Ecto.Multi.run`.

in the end `Ecto.Multi` struct is usually passed to `Repo.transaction/1` which
rollbacks transaction if any operation fails however calling `Repo.transaction/1`
in the end is not obligatory (if you don't want all operations to be run in
transaction - for example, they write to filesystem which cannot be rolled back).

it all resembles using monads to handle errors in general and using
`dry-monads` and `dry-matcher` gems in Ruby in particular
([monads in Ruby]({% post_url 2017-02-27-monads-in-ruby %})).

but unlike `dry-matcher` `Ecto.Multi` stores additional information in case of
failure - not only do we have error itself but also operation name and changes
accumulated in previous succeeded operations.

in this regard `Ecto.Multi` acts more like `dry-transaction` gem which allows to
handle errors arising from particular steps (= operations) with the difference
that `dry-transaction` holds operations (service objects that respond to `call`)
from DI container while multi packs operations (`Repo` or arbitrary functions).

also don't forget that there exist several monad libraries for Elixir
(e.g. [MonadEx](https://github.com/rob-brown/MonadEx)) which allow to use
this error handling mechanism in any module while `Ecto.Multi` is used when
you deal with persistence and need something to replace callbacks
(that is `Ecto.Multi` is alternative to our custom operations in Rails projects
which both persist data and run `after_*` callbacks manually).

<https://github.com/elixir-ecto/ecto/issues/1114#issuecomment-162985202>:

if it's necessary to get access to result of previous operation it's necessary
to use `run` functions - say, if you first insert a record and then need to
updated it you cannot use `Ecto.Multi.update` in the 2nd case since you won't
be able to get previously inserted record.

<https://elixirforum.com/t/ecto-multi-without-transaction/7453>:

it's not possible to execute `Ecto.Multi` operations outside of transaction -
use [with](https://hexdocs.pm/elixir/master/Kernel.SpecialForms.html#with/1)
statement instead.

get query SQL
-------------

1. <https://hexdocs.pm/ecto/Ecto.Adapters.SQL.html#to_sql/3>

```elixir
> import Ecto.Query
> query = from p in User
#Ecto.Query<from u in User>
> Ecto.Adapters.SQL.to_sql(:all, Repo, query)
{"SELECT u0.\"id\", u0.\"name\" FROM \"users\" AS u0", []}
```

[Programming Phoenix] prefixes
------------------------------

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

embeds
------

1. <http://blog.plataformatec.com.br/2015/08/working-with-ecto-associations-and-embeds/>
2. <https://robots.thoughtbot.com/embedding-elixir-structs-in-ecto-models>
3. <https://elixirforum.com/t/cast-embed-causing-error/6678/3>

### updating

you cannot update embed unless you set `on_replace: :update` option:

```elixir
embeds_one :data, TransferData, on_replace: :update
```

otherwise calling `cast_embed(changeset, :data)` will raise error:

```
** (RuntimeError) you are attempting to change relation :data of
Billing.App.Transfer but the `:on_replace` option of
this relation is set to `:raise`.
```

after that you are allowed to pass updated fields as a map only:

```
** (RuntimeError) you have set that the relation :data of Billing.App.Transfer
has `:on_replace` set to `:update` but you are giving it a struct/
changeset to put_assoc/put_change.

Since you have set `:on_replace` to `:update`, you are only allowed
to update the existing entry by giving updated fields as a map or
keyword list or set it to nil.
```

complete example:

```elixir
iex> transfer = Billing.Repo.get(Billing.App.Transfer, 11)
iex> attrs = %{data: %{acs_url: "foo"}}
iex> transfer |> cast(attrs, []) |> cast_embed(:data) |> Repo.update()
```

### nested embeds

1. <https://hexdocs.pm/ecto/Ecto.Schema.html#embeds_one/3-inline-embedded-schema>

_lib/billing/app/transfer.ex_:

```elixir
defmodule Billing.App.Transfer do
  use Ecto.Schema
  import Ecto.Changeset

  alias Billing.App.{Card, TransferData}

  schema "transfers" do
    field :fee, :decimal
    embeds_one :data, TransferData, on_replace: :update

    timestamps type: :utc_datetime
  end

  # > casting embeds with cast/4 is not supported,
  # > use cast_embed/3 instead
  @doc false
  def update_changeset(%__MODULE__{} = transfer, attrs) do
    transfer
    |> cast(attrs, [:fee])
    |> cast_embed(:data)
  end
end
```

_lib/billing/app/transfer_data.ex_:

```elixir
defmodule Billing.App.TransferData do
  use Ecto.Schema
  import Ecto.Changeset

  @primary_key false
  embedded_schema do
    field :acs_url, :string
    embeds_one :error, Error, primary_key: false, on_replace: :update do
      field :service, :integer
    end
  end

  # changeset/2 is called by cast_embed/3 by default
  def changeset(%__MODULE__{} = data, attrs) do
    data
    |> cast(attrs, [:acs_url])
    |> cast_embed(:error, with: &errors_changeset/2)
  end

  defp errors_changeset(%__MODULE__.Error{} = error, attrs) do
    error
    |> cast(attrs, [:service])
  end
end
```

pipe vs. keyword syntax
-----------------------

<https://github.com/elixir-ecto/ecto>:

```elixir
defmodule Sample.App do
  import Ecto.Query
  alias Sample.Weather
  alias Sample.Repo

  def keyword_query do
    query = from w in Weather,
         where: w.prcp > 0 or is_nil(w.prcp),
         select: w
    Repo.all(query)
  end

  def pipe_query do
    Weather
    |> where(city: "Tula")
    |> order_by(:temp_lo)
    |> limit(10)
    |> Repo.all()
  end
end
```
