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

sample data
-----------

1. <https://docs.appsignal.com/elixir/instrumentation/tagging.html>
3. <https://hexdocs.pm/appsignal/Appsignal.Transaction.html#set_sample_data/2>

it's possible to set sample data (aka metadata) for the current transaction
(obviously it must have been started beforehand).

some metadata can be added by AppSignal package automatically - say, request
params when handling request. in other cases we can set sample data manually
with `Appsignal.Transaction.set_sample_data/2` helper function.

sample data can be set under different keys - the name of the key matters as
it determines the way sample data is displayed in AppSignal UI.

### `params` key

as mentioned above `params` key is used by AppSignal to set request params
for the current transaction so it's better to avoid using `params` key when
handling request through a controller action because custom metadata might
override existing one set by AppSignal package.

in other cases (say, in workers) it's very convenient to use this key since
AppSignal will show supplied payload in dedicated `Parameters` section - you
don't have to configure anything in AppSignal for it to work.

`params` key is also special in that `send_params` option is checked before
setting sample data under this key:

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
  Appsignal.Transaction.set_sample_data("params", binding())
  # ...
end
```

note `Kernel.binding/0` returns a keyword list `[user: user, date: date]` -
of course it will be properly serialized before being sent to AppSignal but
using a map is probably a better choice when setting sample data explicitly
(without `Kernel.binding/0`). also map is used to set tags - see below.

***UPDATE (2019-05-20)***

for some reason `Kernel.binding/0` returns empty keyword list in production -
set sample data explicitly using a map:

```elixir
def call(user, date) do
  Appsignal.Transaction.set_sample_data(
    "params",
    %{user: user, date: date}
  )
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

custom metric dashboards
------------------------

### use tags instead of specific name prefixes

1. <https://docs.appsignal.com/metrics/custom.html#metric-tags>
2. <https://docs.appsignal.com/metrics/custom-metrics/dashboards.html#dashboard-graph-metrics-tags>

> <https://docs.appsignal.com/metrics/custom.html#metric-tags>
>
> Custom metrics sometimes need some context what they're about. This
> context can be added as tags so it doesn't need to be included in the
> name and you can use the same metric name for different values.
>
> We do not recommend adding this context to your metric names like so:
> eu.database_size, us.database_size and asia.database_size. This creates
> multiple metrics that serve the same purpose. The same goes for any
> dynamic string that builds the metric key, e.g. user_#{user.id}.

```elixir
# bad
Appsignal.set_gauge("fb.stat.certainty", value)

# good
Appsignal.set_gauge("stat.certainty", value, %{platform: "fb"})
```
