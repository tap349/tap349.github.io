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

timeout vs. recv_timeout
------------------------

> <https://github.com/edgurgel/httpoison/issues/215>
>
> hackney returns {:error, :connect_timeout} when the connect_timeout is
> reached, and {:error, :timeout} when the recv_timeout is reached.
>
> the confusion is because the HTTPoison option :timeout (which corresponds
> to hackney :connect_timeout) returns a :connect_timeout error, whereas the
> HTTPoison :recv_timeout returns the :timeout hackney error.

| HTTPoison option | hackney option  | hackney error   |
|------------------|-----------------|-----------------|
| timeout          | connect_timeout | connect_timeout |
| recv_timeout     | recv_timeout    | timeout         |

```elixir
HTTPoison.get("http://google.com", [], [{:timeout, 1}])
# => {:error, %HTTPoison.Error{id: nil, reason: :connect_timeout}}
HTTPoison.get("http://google.com", [], [{:recv_timeout, 1}])
# => {:error, %HTTPoison.Error{id: nil, reason: :timeout}}
```
