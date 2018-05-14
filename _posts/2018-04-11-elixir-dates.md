---
layout: post
title: Elixir - Dates
date: 2018-04-11 14:06:35 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

Elixir dates
------------

1. <https://hexdocs.pm/elixir/Date.html>

Timex package
-------------

1. <https://hexdocs.pm/timex/basic-usage.html>

### parse string as date

```elixir
"201909"
|> Timex.parse!("%Y%m", :strftime)
|> NaiveDateTime.to_date()
# => ~D[2019-09-01]
```

### format naive datetime as ISO 8601 date

```elixir
~N[2019-09-01 03:00:00]
|> Date.to_iso8601()
# => "2019-09-01"
```

### get beginning/end of month

```elixir
~N[2019-09-01 03:00:00]
|> Timex.beginning_of_month()
# => ~N[2019-09-01 00:00:00]

~N[2019-09-01 03:00:00]
|> Timex.end_of_month()
# => ~N[2019-09-30 23:59:59.999999]
```

### get beginning of next month

```elixir
~N[2019-09-01 03:00:00]
|> Timex.shift(months: 1)
# => ~N[2019-10-01 03:00:00]
```

Ecto dates
----------

1. <https://hexdocs.pm/ecto/Ecto.Schema.html#module-primitive-types>

`Ecto.Date` and `Ecto.DateTime` Ecto types are deprecated and will be removed
in Ecto 3.0 - use Elixir date types instead:

> Since Ecto 2.1, Ecto also supports the Calendar types that are part of
> Elixir standard library:

ECTO TYPE         | ELIXIR TYPE
----------------- | ---------------
`:date`           | `Date`
`:time`           | `Time`
`:naive_datetime` | `NaiveDateTime`
`:utc_datetime`   | `DateTime`

timezones
---------

1. <https://www.amberbit.com/blog/2017/8/3/time-zones-in-postgresql-elixir-and-phoenix/>

migration:

```elixir
defmodule MyApp.Repo.Migrations.CreateUsers do
  use Ecto.Migration

  def change do
    create table("users") do
      # ...

      timestamps(type: :utc_datetime)
    end
  end
end
```

schema:

```elixir
defmodule MyApp.User do
  use Ecto.Schema
  import Ecto.Changeset

  schema "users" do
    # ...

    timestamps(type: :utc_datetime)
  end
end
```
