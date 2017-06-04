---
layout: post
title: Elixir - Testing
date: 2017-06-04 14:25:16 +0300
access: public
categories: [elixir, testing]
---

<!-- more -->

```sh
$ mix test
```

## running specific tests (same as `focus` in RSpec)

- provide test path and line number to `mix test`

  ```sh
  $ mix test test/foo.exs:14
  ```

- use custom tag

  _test/foo.exs_:

  ```elixir
  @tag :wip
  test "run this test only" do
    assert true
  end
  ```

  ```sh
  $ mix test --only wip
  ```

## using IEx.pry in tests

to make pry work in tests run `mix test` in IEx session:

```sh
$ iex -S mix test
```

NOTE: in my case `iex` is aliased to `iex -S mix` so type just `iex test`.
