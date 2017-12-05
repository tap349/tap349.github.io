---
layout: post
title: Elixir - HTTPoison
date: 2017-08-18 02:22:15 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

submit form data
----------------

1. <https://github.com/edgurgel/httpoison/blob/master/test/httpoison_test.exs>

```elixir
{:ok, %HTTPoison.Response{body: body}} = HTTPoison.post(
  "http://example.com",
  {:form, [key_1: "value_1", key_2: "value_2"]},
  %{"Content-Type" => "application/x-www-form-urlencoded"}
)
```

NOTE: `Content-Type` header is optional because we have
      explicitly specified that we are posting form data.
