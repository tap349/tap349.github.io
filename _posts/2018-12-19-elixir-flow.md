---
layout: post
title: Elixir - Flow
date: 2018-12-19 02:01:00 +0300
access: private
comments: true
categories: [elixir, flow]
---

<!-- more -->

* TOC
{:toc}
<hr>

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

=> number of stages = number of CPU cores by default

say, on my MacBook with 4 cores:

```elixir
1..10
|> Flow.from_enumerable(max_demand: 1)
|> Flow.map(fn x -> IO.inspect(x); Process.sleep(2000); x + 100 end)
|> Enum.into([])
```

prints

```elixir
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
