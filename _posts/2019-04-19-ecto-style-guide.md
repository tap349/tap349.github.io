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
