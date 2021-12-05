---
layout: post
title: Elixir - Dates
date: 2018-04-11 14:06:35 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://hexdocs.pm/elixir/Date.html>
2. <https://hexdocs.pm/timex/basic-usage.html>

tips
----

### parse string in arbitrary format as date

```elixir
"201909"
|> Timex.parse!("%Y%m", :strftime)
|> NaiveDateTime.to_date()
# => ~D[2019-09-01]
```

### parse ISO 8601 string as datetime

1. <https://hexdocs.pm/timex/Timex.html#parse!/2>
2. <https://hexdocs.pm/elixir/DateTime.html#from_iso8601/2>
3. <https://hexdocs.pm/elixir/DateTime.html#from_naive!/2>
4. <https://hexdocs.pm/elixir/NaiveDateTime.html#from_iso8601!/2>

```elixir
Timex.parse!("2018-08-28T08:42:29+0000", "{ISO:Extended}")
# => #DateTime<2018-08-28 08:42:29Z>

DateTime.from_iso8601("2018-08-28T08:42:29+0000")
# => {:ok, #DateTime<2018-08-28 08:42:29Z>, 0}

DateTime.from_naive!(
  NaiveDateTime.from_iso8601!("2018-08-28T08:42:29+0000"),
  "Etc/UTC"
)
# => #DateTime<2018-08-28 08:42:29Z>
```

### format naive datetime as ISO 8601 date

1. <https://hexdocs.pm/elixir/Date.html#to_iso8601/2>

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

### compare dates

```elixir
datetime_1 = Timex.parse!("2018-08-30 01:00:00.000000Z", "{ISO:Extended}")
datetime_2 = Timex.parse!("2018-08-30 01:00:01.000000Z", "{ISO:Extended}")

Timex.before?(datetime_1, datetime_2)
# => true
```

don't use comparison operators (`>`, `>=`, etc.) when comparing dates:

> <https://elixirforum.com/t/elixirs-datetime-values-dont-seem-to-compare-correctly/3243>
>
> Basic Erlang ordering often produces confusing results when you apply
> it to complicated data structures such as a map with Tuples.
>
> Maps are ordered by size, two maps with the same size are compared by
> keys in ascending term order and then by values in key order. In maps
> key order integers types are considered less than floats types.
>
> So since DateTime is a map, it will sort the keys in term order, and
> microsecond is before minute.

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
