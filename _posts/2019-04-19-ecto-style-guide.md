---
layout: post
title: Ecto - Style Guide
date: 2019-04-19 16:48:23 +0300
access: public
comments: true
categories: [elixir, ecto]
---

<!-- more -->

* TOC
{:toc}
<hr>

naming conventions
------------------

### mutator functions

- `create_*` functions call `Ecto.Repo.insert` inside
- `import_*` functions call `Ecto.Repo.insert_all` inside

### permitted parameters

- `@create_permitted_params`
- `@update_permitted_params`

order
-----

### order of schema fields

association field comes last in:

- schemas

  ```elixir
  defmodule Post do
    schema "posts" do
      field :title, :string
      belongs_to :user, User
    end
  end
  ```

- test factories

  ```elixir
  defmodule Factory do
    def build(:post) do
      %Post{
        title: "foo",
        user: build(:user)
      }
    end
  end
  ```

- Ecto queries

  ```elixir
  Post
  |> where(title: ^title)
  |> where(user_id: ^user_id)
  |> Repo.all()
  ```

association field comes first in:

- function definitions

  ```elixir
  defmodule Post.Loader do
    def get_by_user_and_title(user_id, title) do
      Post
      |> where(title: ^title)
      |> where(user_id: ^user_id)
      |> Repo.all()
    end
  end
  ```

  rationale: even though it's Ecto's choice to always place association after
  other schema fields, it seems more natural to pass association first from a
  client perspective.

migrations
----------

### always add `null: true` option in migrations

1. <https://hexdocs.pm/ecto_sql/Ecto.Migration.html#modify/3>
2. <https://www.postgresql.org/docs/11/ddl-constraints.html#id-1.5.4.5.6>

Ecto doesn't add not-null constraint by default when adding new column but
specifying `null: true` option explicitly is required when modifying column
to drop existing not-null constraint (or else it won't be dropped).

so always add `null: true` in migrations for the sake of consistency.

tests
-----

### default values

- test factories

  string fields:

  - `John Doe`/`Jane Doe` (or just `John`/`Jane`) for name fields
  - `example.com` for domain fields
  - some common value for enum fields (`currency: "USD"`)
  - field name as its default value (`login: "login"`)

    it can be suffixed by some ID (say, `ext_id` if available)

  integer fields:

  - sequence of integer values starting from 1
  - automatically generated integer (`System.unique_integer([:positive])`) for
    fields with unique index on them

- API stubs

  use the same default values as in test factories.

- tests

  it's allowed to use any values including notorious `foo`/etc.

### full vs. partial API result

as a rule JSON API result is either an object (hash, map) or an array of such
objects.

- API tests

  - use full objects (with all fields left) in API tests
  - don't sort fields (in alphabetical order or in order of significance)
  - if API result is an array of objects, use the first object only:

    ```elixir
    assert [%{"foo" => 123} | _] = result
    ```

- API stubs and non-API tests

  - use partial objects (with some fields dropped)
  - leave only the fields which are used in business logic and remove the others
  - don't sort fields either

### asynchronous database tests

1. <https://hexdocs.pm/phoenix/testing_schemas.html#test-driving-a-changeset>

<http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html>:

> By default, Phoenix uses the Ecto.Adapters.SQL.Sandbox module. Which
> basically wraps every test within a transaction in order to rollback
> it after a test is finished. That helps to keep the test database clean.
>
> "Testing Schemas" guide does not recommend us to use `async: true` option
> if we are going to interact with the database:
>
> > Note: We should not tag any schema case that interacts with a database
> > as :async. This may cause erratic test results and possibly even deadlocks.
>
> Although, Ecto.Adapters.SQL.Sandbox also contains a note about that:
>
> > While both PostgreSQL and MySQL support SQL Sandbox, only PostgreSQL
> supports concurrent tests while running the SQL Sandbox. Therefore, do
> not run concurrent tests with MySQL as you may run into deadlocks due
> to its transaction implementation.
>
> We are using PostgreSQL for that project. So I am going to enable that
> option at my own peril.
