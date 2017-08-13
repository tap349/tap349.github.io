---
layout: post
title: Elixir - Debugging
date: 2017-07-30 15:49:05 +0300
access: public
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

### using `IEx.pry` (same as `binding.pry` in Ruby)

- <http://blog.plataformatec.com.br/2016/04/debugging-techniques-in-elixir-lang/>
- <https://stackoverflow.com/questions/29671156/pry-while-testing>

- add `IEx.pry` breakpoint

  ```elixir
  require IEx

  defmodule Test do
    def foo do
      IEx.pry
    end
  end
  ```

- start new IEx session or recompile module with breakpoint
- finish pry session by calling `respawn`

### debugging dependencies

sometimes it might be necessary to debug or temporarily change external
dependencies stored in _deps/_ directory:

```sh
$ mvim deps/detergentex/lib/detergentex.ex
$ mix deps.compile detergentex
$ mix -S iex
```

### information about endpoint

1. <https://elixirforum.com/t/how-can-i-see-what-port-a-phoenix-app-in-production-is-actually-trying-to-use/5160/5>

```sh
$ bin/billing remote_console
iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint.Server
iex(billing@127.0.0.1)1> :sys.get_state BillingWeb.Endpoint
```
