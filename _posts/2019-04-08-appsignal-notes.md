---
layout: post
title: AppSignal - Notes
date: 2019-04-08 15:36:18 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

log location
------------

1. <https://docs.appsignal.com/elixir/configuration/options.html#option-working_directory_path>

> <https://docs.appsignal.com/support/debugging.html#log-location>
>
> The Elixir package will write the log files to the /tmp/appsignal.log
> log-file.

that is AppSignal log is not created in `working_directory_path` as I expected.

setting sample data
-------------------

1. <https://docs.appsignal.com/elixir/instrumentation/tagging.html>
3. <https://hexdocs.pm/appsignal/Appsignal.Transaction.html#set_sample_data/2>

it's possible to add sample data (aka metadata) for the current transaction
(obviously it must have been started beforehand).

sample data can be set with `Appsignal.Transaction.set_sample_data/2` under
different keys.

### `params` key

in theory any key can be used but it's better to use `params` key - in this
case AppSignal will show supplied payload in dedicated `Parameters` section
(the same key and section must be used for controller request params - avoid
using `params` key when handling a request through a controller action since
custom metadata may override existing one set by AppSignal package).

note that `params` key is also special in that `send_params` option is checked
before setting sample data under this key:

```elixir
# https://github.com/appsignal/appsignal-elixir/blob/master/lib/appsignal/transaction.ex#L263

def set_sample_data(%Transaction{} = transaction, "params", payload) do
  if config()[:send_params] do
    do_set_sample_data(transaction, "params", payload)
  else
    transaction
  end
end
```

example:

```elixir
def call(user, date) do
  # binding/0 returns a keyword list [user: <USER>, date: <DATE>] -
  # it will be properly serialized when being sent to AppSignal
  Appsignal.Transaction.set_sample_data("params", binding())

  # ...
end
```

### `tags` key

1. <https://docs.appsignal.com/application/link-templates.html>

`tags` key is used to pass tags which can be used later to create links with
link templates:

> <https://docs.appsignal.com/application/link-templates.html>
>
> For link templates to work AppSignal needs to have the data necessary
> to create the links. Start by adding tags to request in your application.

AppSignal will show these links in `Overview` section.

### other keys

IDK how to display sample data set under different keys in AppSignal UI.
