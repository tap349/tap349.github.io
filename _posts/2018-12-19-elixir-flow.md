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

> <https://www.youtube.com/watch?v=srtMWzyqdp8>
>
> max_demand: the maximum amount of events to ask (default 1000)
> min_demand: when reached, ask for more events (default 500)

=> batch size = max_demand - min_demand

> <https://hexdocs.pm/flow/Flow.html#from_enumerable/2>
>
> The enumerable is consumed in batches, retrieving max_demand items
> the first time and then max_demand - min_demand the next times.
