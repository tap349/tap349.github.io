---
layout: post
title: Elixir - AppSignal
date: 2018-09-03 02:10:40 +0300
access: public
comments: true
categories: [elixir, appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

AppSignal transaction
---------------------

1. <https://docs.appsignal.com/elixir/instrumentation/instrumentation.html>
2. <https://github.com/appsignal/appsignal-elixir/issues/299>

transaction is created automatically for HTTP requests only (when Phoenix
integration is set up) - all workers must start transaction manually, say,
using AppSignal decorators:

```elixir
defmodule MyApp.Worker
  use Appsignal.Instrumentation.Decorators

  @decorate transaction(:background_job)
  def call do
    # ...
  end
end
```

if AppSignal transaction is not started (say, in GenServer):

- instrumentation is disabled

  > <https://docs.appsignal.com/elixir/instrumentation/instrumentation.html#decorator-transactions>
  >
  > In order to track transaction_event decorators we will need to start
  > an AppSignal transaction beforehand.

  performance data is sent inside AppSignal transaction only.

- unhandled errors are sent

  e.g. when error is raised manually with `raise/1`.

  they are ALWAYS sent. no need to wrap workers in AppSignal transaction,
  Phoenix integration might be disabled as well - it just works, IDK why.

  however unhandled errors are not sent when function with `transaction`
  decorator is called manually in IEx.

- `Appsignal.Transaction.set_error/3` doesn't work

  1. <https://docs.appsignal.com/elixir/instrumentation/exception-handling.html#appsignal-transaction-set_error-3>

  it sends errors inside AppSignal transaction only.

- `Appsignal.send_error/6` works

  1. <https://docs.appsignal.com/elixir/instrumentation/exception-handling.html#appsignal-send_error-6>

  it always sends errors irrespective of AppSignal transaction.
