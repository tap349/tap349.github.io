---
layout: post
title: Elixir - Flow
date: 2018-12-19 02:01:00 +0300
access: public
comments: true
categories: [elixir, flow]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://www.youtube.com/watch?v=srtMWzyqdp8>
2. <https://10consulting.com/2017/01/20/building-product-recommendations-using-elixir-gen-stage-flow>
3. <https://nickstalter.com/elixir-flow.html>
4. <http://www.elixirfbp.org/2016/11/genstage-and-bioinformatics.html>

example projects using Flow:

1. <https://github.com/nietaki/crawlie>

eager (`Enum`) → lazy (`Stream`) → concurrent (`Flow`) → distributed (Apache Spark)

so `Flow` brings computations on collections to the next level as it allows
to process collections not only lazily but concurrently as well - the level
of concurrency is determined by the number of mapper and reducer stages.

=> `Stream` allows to limit memory usage, `Flow` allows to reduce processing
time.

max_demand vs. min_demand
-------------------------

> <https://hexdocs.pm/flow/Flow.html>
>
> By default, Flow will work with batches of 500 items.

> <https://www.youtube.com/watch?v=srtMWzyqdp8>
>
> max_demand: the maximum amount of events to ask (default 1000)
> min_demand: when reached, ask for more events (default 500)

> <https://hexdocs.pm/flow/Flow.html#from_enumerable/2>
>
> The enumerable is consumed in batches, retrieving max_demand items
> the first time and then max_demand - min_demand the next times.

most likely `min_demand` is the number of *unprocessed* events in consumer
(aka stage).

say, if `max_demand` = 100 and there remains 90 unprocessed events, consumer
will ask producer for the next (100 - 90) = 10 events. so high `min_demand`
results in small batches which is perfectly fine for slow consumers as this
allows more stages to consume remaining events concurrently while in case of
large batches there's a risk that all these events would be sent to the same
stage or a small number of stages.

number of stages
----------------

> <http://www.akitaonrails.com/2017/06/13/ex-manga-downloadr-part-6-the-rise-of-flow#comment-3360301947>
>
> Another parameter you usually tune is the `stages: ...` option, you should
> set that to the amount of connections you had in poolboy in the past.

> <https://hexdocs.pm/flow/Flow.html#module-configuration-demand-and-the-number-of-stages>
>
> If stages perform IO, it may also be worth increasing the number of stages.
> The default value is System.schedulers_online/0, which is a good default if
> the stages are CPU bound, but if stages are waiting on external resources or
> other processes, increasing the number of stages may be helpful.

=> number of stages = number of CPU cores by default.

say, on my MacBook with 4 cores:

```elixir
1..10
|> Flow.from_enumerable(max_demand: 1)
|> Flow.map(fn x -> IO.inspect(x); Process.sleep(2000); x + 100 end)
|> Enum.into([])

# =>

1
2
3
4
# sleep
5
6
7
8
# sleep
9
10
[101, 102, 103, 104, 105, 106, 107, 108, 109, 110]
```

### number of stages with and without partitioning

without partitioning the same stages are reused for all Flow operations.
I want to stress it once again: all Flow operations in the pipe are made
in the same stages (= the same processes).

for each collection item `Flow.flat_map/2` and `Flow.reduce/3` operations
are executed sequentially and in the same stage:

```elixir
["foo", "bar"]
|> Flow.from_enumerable()
|> Flow.flat_map(...)
|> Flow.reduce(...)
```

=>

```
[file stream]  # Flow.from_enumerable/1 (producer)
   |    |
 [M1]  [M2]    # Flow.flat_map/2 + Flow.reduce/3 (consumer)
```

partitioning always creates new stages:

```elixir
["foo", "bar"]
|> Flow.from_enumerable()
|> Flow.flat_map(...)
|> Flow.partition()
|> Flow.reduce(...)
```

=>

```
[file stream]  # Flow.from_enumerable/1 (producer)
   |    |
 [M1]  [M2]    # Flow.flat_map/2 (producer-consumer)
   |\  /|
   | \/ |
   |/ \ |
 [R1]  [R2]    # Flow.reduce/3 (consumer)
```

partitioning doesn't have to keep the same number of stages as before:

```elixir
["foo", "bar", "baz", "foo"]
|> Flow.from_enumerable(stages: 4, max_demand: 1)
|> Flow.partition(5)
|> Flow.uniq()
|> Enum.to_list()
```

in this case there are 4 stages before partitioning and 5 stages after it.
default hashing function routes both `foo` words to the same partition so
in fact only 3 stages will be used and 2 stages will remain idle.

### another example with and without partitioning

```elixir
fetch_sitemap_shop_urls()
|> Flow.from_enumerable(stages: 57, max_demand: 1)
|> Flow.flat_map(&fetch_shop_urls/1)
|> Flow.each(&import_shops/1)
|> Flow.run()
```

=> collection items are processed using 57 stages concurrently (each
stage receives only one collection item because of `max_demand: 1`).

```elixir
fetch_sitemap_shop_urls()
|> Flow.from_enumerable(stages: 57, max_demand: 1)
|> Flow.flat_map(&fetch_shop_urls/1)
|> Flow.partition()
|> Flow.each(&import_shops/1)
|> Flow.run()
```

=> collection items are processed using 57 stages before `Flow.partition/2`
and using 4 stages after `Flow.partition/2`. by and large 57 + 4 processes
will be created (+1 process for `Flow.from_enumerable/1`).

max_demand vs. number of stages
-------------------------------

it's important to note that `max_demand` items constitute a single batch,
batch items are always sent to the same stage one by one. that is producer
sends the first `max_demand` events to the 1st stage, the next `max_demand`
events - to the 2nd stage and so on.

if there remains less than `max_demand` events in enumerable, they are sent
to the next stage. if there are any unused stages left, they remain idle for
ever and do nothing at all.

say, collection size = 100, `max_demand` = 90, `stages` = 4: Flow sends the
first 90 events to the 1st stage and remaining 10 events - to the 2nd stage.
the other 2 stages remain idle. as a result the first 2 stages process their
first 10 events concurrently and then 80 events are processed synchronously
by the 1st stage.

> <http://www.akitaonrails.com/2017/06/13/ex-manga-downloadr-part-6-the-rise-of-flow#comment-3360301947>
>
> @max_demand is how much data you exchange between stages. If you set it to
> 60, it means you are sending 60 items to one stage, 60 items to the other
> and so on. That gives you poor balancing for small collections as there is
> a chance all items end-up in the same stage. You actually want to reduce
> `max_demand` to 1 or 2 so each stage gets small batches and request more
> than needed.

```elixir
iex> 1..10
|> Flow.from_enumerable(max_demand: 3)
|> Flow.map(fn x -> IO.inspect(x); Process.sleep(2000); x + 100 end)
|> Enum.into([])

# =>

1
4
7
10
# sleep
2
5
8
# sleep
6
3
9
[110, 104, 105, 101, 102, 107, 108, 106, 103, 109]
```

### max_demand items in the pipe

it's important to note that there can be max `max_demand` items in the pipe
at any given moment - that is in one stage because each item is processed by
multiple operations in the same stage (if there is no partitioning).

say, there are 2 Flow operations in the pipe:

- `max_demand` is 1

  the 2nd item can be processed by the 1st operation only after the 1st item
  is processed by both operations => there is only 1 item in the pipe at any
  given moment.

- `max_demand` is 2

  the 2nd item can be processed by the 1st operation only after the 1st item
  is processed by the 1st operation but the the 3rd item cannot be processed
  by the 1st operation until the 1st item is processed by both operations =>
  there are only 2 items in the pipe at any given moment.

most likely partitioning breaks the pipe into 2 parts - before and after it.
so 2 Flow operations are executed in different stages which have their own
`max_demand`. I guess new parts act as standalone pipes and the rule above
applies to each of them independently => even if `max_demand` is 1, new item
can be consumed by the 1st stage when the 1st item reaches a reducer stage.

***UPDATE***

maybe this is not true: with `max_demand: 2` option processing of the 2nd item
didn't start right after the 1st item was processed by the 1st operation (there
is no partitioning).

partitioning with Flow.partition/2
----------------------------------

1. <https://hexdocs.pm/flow/Flow.html#partition/2>
2. <http://blog.plataformatec.com.br/2018/07/whats-new-in-flow-v0-14>
3. <https://10consulting.com/2017/01/20/building-product-recommendations-using-elixir-gen-stage-flow>

see the full article for complete example of using Flow to count words and
using `Flow.partition/2` in particular:

> <http://blog.plataformatec.com.br/2018/07/whats-new-in-flow-v0-14>
>
> The key operation in the example above is precisely the partition/2 call.
> Since we want to count words, we need to make sure that we will always
> route the same word to the same partition, so all occurrences belong to
> a single place and not scattered around.

### stages and max_demand

`Flow.partition/2` doesn't respect previous values of `stages` and
`max_demand` options:

```elixir
list
|> Flow.from_enumerable(stages: 10) # 10 stages
|> Flow.partition() # System.schedulers_online() stages
```

```elixir
list
|> Flow.from_enumerable(max_demand: 10) # max_demand = 10
|> Flow.partition() # max_demand = 1 (?)
```

it looks like `Flow.partition/2` sets `max_demand` to 1 unless specified
explicitly even though `max_demand` defaults to 1000 according to Flow docs.

error handling
--------------

1. <https://elixirforum.com/t/flow-error-handling/7473/3>

it looks like if error is raised in one of stages, all other stages complete
processing of their current events (max `max_demand` events in each stage) -
still current events in the stage where error is raised are left unprocessed.

at the same time no new events are accepted for processing - that is further
processing of input collection is immediately blocked but Flow waits till all
current events are processed and exits only after that:

```
22:31:43.192 [error] Task #PID<0.821.0> started from #PID<0.786.0> terminating
** (stop) exited in: GenStage.close_stream(%{
      #Reference<0.1001417396.713031682.201732> => {:subscribed, #PID<0.826.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201733> => {:subscribed, #PID<0.827.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201734> => {:subscribed, #PID<0.828.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201735> => {:subscribed, #PID<0.829.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201736> => {:subscribed, #PID<0.830.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201737> => {:subscribed, #PID<0.831.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201738> => {:subscribed, #PID<0.832.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201739> => {:subscribed, #PID<0.833.0>, :transient, 500, 1000, 1000},
      #Reference<0.1001417396.713031682.201740> => {:subscribed, #PID<0.834.0>, :transient, 500, 1000, 1000}
    })
    ** (EXIT) an exception was raised:
        ** (RuntimeError) Error fetching http://example.com/sitemap.xml: closed
```

as a result input collection is processed only partially.
