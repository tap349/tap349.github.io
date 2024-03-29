---
layout: post
title: Ecto - Style Guide
date: 2019-04-19 16:48:23 +0300
access: public
comments: true
categories: [elixir, ecto]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

## naming conventions

### mutator functions

- `create_*` functions call `Ecto.Repo.insert` inside
- `import_*` functions call `Ecto.Repo.insert_all` inside

### permitted parameters

- `@create_permitted_params`
- `@update_permitted_params`

## order

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

## migrations

### always add `null: true` option in migrations

1. <https://hexdocs.pm/ecto_sql/Ecto.Migration.html#modify/3>
2. <https://www.postgresql.org/docs/11/ddl-constraints.html#id-1.5.4.5.6>

Ecto doesn't add not-null constraint by default when adding new column but
specifying `null: true` option explicitly is required when modifying column to
drop existing not-null constraint (or else it won't be dropped).

so always add `null: true` in migrations for the sake of consistency.
