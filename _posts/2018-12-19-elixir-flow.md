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

max_demand vs. min_demand
-------------------------

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
of small collections and large batches these events might have been reserved
for a single stage.

number of stages
----------------

> <https://hexdocs.pm/flow/Flow.html#module-partitioning>
>
> This will execute the flat_map and reduce operations in parallel
> inside multiple stages. When running on a machine with two cores:
>
> [file stream]  # Flow.from_enumerable/1 (producer)
>    |    |
>  [M1]  [M2]    # Flow.flat_map/2 + Flow.reduce/3 (consumer)

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

max_demand vs. number of stages
-------------------------------

it's important to note that `max_demand` items constitute a single batch and
are sent to the same stage: producer sends the first `max_demand` events to
the 1st stage, the next `max_demand` events - to the 2nd stage and so on.

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
