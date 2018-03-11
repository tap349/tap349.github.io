---
layout: post
title: Elixir - ETS
date: 2018-03-11 17:56:03 +0300
access: public
comments: true
categories: [elixir, ets]
---

<!-- more -->

* TOC
{:toc}
<hr>

<https://elixirforum.com/t/data-caching-agents-or-ets/1614/8> (Sasa Juric):

> Agent is a simple solution that could work for smaller loads and a few client
> processes. ETS table should usually perform better, and can support concurrent
> clients, i.e. you could have simultaneous multiple readers/writers - something
> not possible with Agent/GenServer. It is however very limited in terms of atomic
> operations, so it’s mostly suitable for simple k-v stuff, and some concurrent counters.
>
> Personally, if I know that there will be multiple clients of a key-value store,
> I just go for ETS immediately, because I believe this is what it was made for.
> That being said, some cases are in the grey area, so starting with a simple Agent
> is a somewhat simpler and more flexible solution. Assuming you encapsulate cache
> operations with some module, switching to ETS should be easy, because you’ll
> likely need to change the implementation in only one module (the cache wrapper).
>
> Finally, as others have pointed out, think carefully whether you even need a
> cache. All other things being equal, cacheless is better than cacheful (because
> of less complexity), so if you can get away without it, it will be the simplest
> solution.
