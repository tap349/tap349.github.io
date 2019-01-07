---
layout: post
title: Phoenix - Style Guide
date: 2018-03-26 13:36:31 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

naming
------

### attrs vs. params

`params` is what comes from controller - they might map to `attrs` exactly or
not => use `params` everywhere except for cases when it's clear that you deal
with attributes - say, when you create a map of attributes by yourself or in
changeset functions inside schema modules.

NOTE: in Phoenix and Ecto docs they now use `params` everywhere.

***UPDATE***

<https://hexdocs.pm/phoenix/ecto.html#the-schema>:

```elixir
def changeset(%User{} = user, attrs) do
  user
  |> cast(attrs, [:name, :email, :bio, :number_of_pets])
  |> validate_required([:name, :email, :bio, :number_of_pets])
end
```

<https://hexdocs.pm/ecto/Ecto.Schema.html#many_to_many/3-join-schema-example>:

```elixir
def changeset(struct, params \\ %{}) do
  struct
  |> Ecto.Changeset.cast(params, [:user_id, :organization_id])
  |> Ecto.Changeset.validate_required([:user_id, :organization_id])
  # Maybe do some counter caching here!
end
```

<https://hexdocs.pm/ecto/Ecto.Changeset.html>:

```elixir
def changeset(user, params \\ %{}) do
  user
  |> cast(params, [:name, :email, :age])
  |> validate_required([:name, :email])
  |> validate_format(:email, ~r/@/)
  |> validate_inclusion(:age, 18..100)
  |> unique_constraint(:email)
end
```

=> in most cases `params` are used.

config
------

common sensible configuration should take place in _config/config.exs_ -
it can be amended with environment-specific details in environment config
files (_config/dev.exs_, etc.).

when in doubt place production configuration in _config/config.exs_ but
without production secrets and details (say, production URLs) that would
prevent application from running in development in case _config/dev.exs_
is removed at all.
