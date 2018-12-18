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

=> batch size = max_demand - min_demand

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

=> number of stages = number of CPU cores by default?
