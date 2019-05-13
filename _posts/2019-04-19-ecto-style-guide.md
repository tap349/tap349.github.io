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

add `null: true` option in migrations
-------------------------------------

1. <https://hexdocs.pm/ecto_sql/Ecto.Migration.html#modify/3>
2. <https://www.postgresql.org/docs/11/ddl-constraints.html#id-1.5.4.5.6>

Ecto doesn't add not-null constraint by default when adding new column but
specifying `null: true` option explicitly is required when modifying column
to drop existing not-null constraint (or else it won't be dropped).

so always add `null: true` in migrations for the sake of consistency.

default values in tests
-----------------------

- test factories

  string fields:

  - `John Doe`/`Jane Doe` (or just `John`/`Jane`) for name fields
  - `example.com` for domain fields
  - some common value for enum fields (`currency: "USD"`)
  - field name as its default value (`login: "login"`)

  integer fields:

  - sequence of integer values starting from 1
  - automatically generated integer (`System.unique_integer([:positive])`) for
    fields with unique index on them

- API stubs

  use the same default values as in test factories.

- tests

  it's allowed to use any values including notorious `foo`/etc.

naming conventions
------------------

### in operations

- `create_*` functions call `Ecto.Repo.insert` inside
- `import_*` functions call `Ecto.Repo.insert_all` inside

### in schemas

- `@create_permitted_params`
- `@update_permitted_params`

order of schema fields
----------------------

association field always comes last in:

- schema

  ```elixir
  defmodule Post do
    schema "posts" do
      field :title, :string
      belongs_to :user, User
    end
  end
  ```

- factory

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

- schema loader

  ```elixir
  defmodule Post.Loader do
    def get_by_title_and_user(title, user_id) do
      Post
      |> where(title: ^title)
      |> where(user_id: ^user_id)
      |> Repo.all()
    end
  end
  ```

on the other hand if this is user loader (operation, service), user will always
come first even if it will be passed to post loader as a last argument later:

```elixir
defmodule User.Loader do
  def get_all_post_titles(user, title) do
    title
    |> Post.Loader.by_title_and_user(user.id)
    |> Enum.map(& &1.title)
  end
end
```

