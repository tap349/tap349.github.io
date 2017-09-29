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

## run specific tests (same as `focus` in RSpec)

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

## show detailed information (same as `--format documentation` in RSpec)

```sh
$ mix test --trace
```

`--trace` option also sets timeout to infinity - useful when using `IEx.pry`.

## run tests synchronously

`async: true` has no effect - useful in case of race conditions.

```sh
$ mix test --trace
```

## use IEx.pry in tests

to make pry work in tests run `mix test` in IEx session:

```sh
$ iex -S mix test
```

in my case `iex` is aliased to `iex -S mix` so type just `iex test`.

NOTE: test will time out after 60000ms by default:

```
** (ExUnit.TimeoutError) test timed out after 60000ms. You can change the timeout:

  1. per test by setting "@tag timeout: x"
  2. per case by setting "@moduletag timeout: x"
  3. globally via "ExUnit.start(timeout: x)" configuration
  4. or set it to infinity per run by calling "mix test --trace"
     (useful when using IEx.pry)

Timeouts are given as integers in milliseconds.
```
