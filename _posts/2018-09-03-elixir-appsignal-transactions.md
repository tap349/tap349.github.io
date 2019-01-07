---
layout: post
title: Elixir - AppSignal Transactions
date: 2018-09-03 02:10:40 +0300
access: public
comments: true
categories: [elixir, appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://docs.appsignal.com/elixir/instrumentation/instrumentation.html>
2. <https://github.com/appsignal/appsignal-elixir/issues/299>

transaction is created automatically for HTTP requests only (when Phoenix
integration is set up) - all workers must start transaction manually using
function decorators or corresponding helper functions:

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

  => `transaction_event` decorator doesn't work - performance data is sent
  inside AppSignal transaction only.

- unhandled errors are still sent

  e.g. when error is raised manually with `raise/1`.

  they are ALWAYS sent. no need to wrap workers in AppSignal transaction,
  Phoenix integration might be disabled as well - it just works, IDK why.

  however unhandled errors are not sent when function with `transaction`
  decorator is called manually in IEx.

- `Appsignal.Transaction.set_error/3` doesn't work

  1. <https://docs.appsignal.com/elixir/instrumentation/exception-handling.html#appsignal-transaction-set_error-3>

  it sends errors inside AppSignal transaction only.

- `Appsignal.send_error/7` works

  1. <https://docs.appsignal.com/elixir/instrumentation/exception-handling.html#appsignal-send_error-7>

  it always sends errors irrespective of AppSignal transaction.

transactions and child processes
--------------------------------

information about current transaction is lost in child processes!

```elixir
defmodule Worker do
  @decorate transaction(:background_job)
  def perform do
    IO.inspect(Appsignal.TransactionRegistry.lookup(self()))
    # => AppSignal.Transaction{o1m4adl8kaao0}
    task = Task.async(fn ->
      IO.inspect(Appsignal.TransactionRegistry.lookup(self()))
      # => nil
    end)

    Task.await(task)
  end
end
```

there are 2 solutions to use AppSignal in child processes (to add custom
instrumentation and set errors):

- start a new transaction within a child process

  ```elixir
  defmodule Worker do
    use Appsignal.Instrumentation.Decorators

    @decorate transaction(:background_job)
    def perform do
      task = Task.async(fn -> Operation.call() end)
      Task.await(task)
    end
  end

  defmodule Operation do
    use Appsignal.Instrumentation.Decorators

    @decorate transaction(:operation)
    def call do
      fetch_data()
      set_error()
    end

    @decorate transaction_event("fb_api")
    defp fetch_data do
      FB.API.fetch_data()
    end

    defp set_error do
      Appsignal.Transaction.set_error("RuntimeError", "foo", [])
    end
  end
  ```

- pass parent PID or parent transaction PID explicitly

  pass parent PID to add custom instrumentation:

  > <https://docs.appsignal.com/elixir/instrumentation/instrumentation.html#adding-asynchronous-events-to-a-transaction>
  >
  > To add events to another process' transaction you can pass along the PID of
  > a process to the instrument/4 function. If a transaction exists for that
  > process, the event will be registered on that transaction, otherwise it's
  > ignored.

  ```elixir
  defmodule Worker do
    use Appsignal.Instrumentation.Decorators
    import Appsignal.Instrumentation.Helpers, only: [instrument: 4]

    @decorate transaction(:background_job)
    def perform do
      parent = self()

      task = Task.async(fn ->
        # = @decorate transaction_event("fb_api")
        instrument(parent, "fb_api", "Fetching FB data", fn ->
          FB.API.fetch_data()
        end)
      end)

      Task.await(task)
    end
  end
  ```

  pass parent transaction PID to set errors:

  ```elixir
  defmodule Worker do
    def perform do
      transaction = Appsignal.Transaction.start(
        Appsignal.Transaction.generate_id(),
        :background_job
      )

      task = Task.async(fn ->
        set_error(transaction, "RuntimeError", "foo", [])
      end)

      Task.await(task)
    end
  end
  ```

tips
----

- get current transaction

  ```elixir
  Appsignal.TransactionRegistry.lookup(self())
  # => AppSignal.Transaction{o1m4adl8kaao0}
  ```

  this is how to choose between using `set_error` and `send_error`:

  ```elixir
  case Appsignal.TransactionRegistry.lookup(self()) do
    nil -> Appsignal.send_error(%RuntimeError{}, message, [])
    _ -> Appsignal.Transaction.set_error("RuntimeError", message, [])
  end
  ```
