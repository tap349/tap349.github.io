---
layout: post
title: Elixir - Process Pooling
date: 2018-12-25 13:58:52 +0300
access: public
comments: true
categories: [poolboy, flow]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

there are several solutions which allow to process collections concurrently
and to keep a constant amount of processes (pool) running at any given time.

- `Task.async_stream` and `Task.async_stream_nolink`

  1. <https://hexdocs.pm/elixir/Task.html#async_stream/5>
  2. <https://hexdocs.pm/elixir/Task.Supervisor.html#async_stream/6>
  3. <http://www.akitaonrails.com/2017/06/16/ex-manga-downloadr-part-7-properly-dealing-with-large-collections>

  each process is called a task (implemented as `Task`).

  this is the simplest solution which however must be sufficient for most
  use cases. what is important is that it's built into the language - you
  don't need to add another dependency.

  > <http://www.akitaonrails.com/2017/06/13/ex-manga-downloadr-part-6-the-rise-of-flow#comment-3360301947>
  >
  > Elixir v1.4 has something called `Task.async_stream` which is a mixture of
  > poolboy + task async. We have added it to Elixir after implementing Flow as
  > we realized you can get a lot of mileage out of `Task.async_stream` without
  > needing to jump to a full solution like Flow. If using `Task.async_stream`,
  > the `max_concurrency` option controls your pool size.

- [Poolboy]({% post_url 2017-12-12-elixir-poolboy %})

  each process is called a worker (implemented as `GenServer`).

  `Poolboy` is a battle-tested Erlang pooling solution but still it's a
  third-party library and hence one more depenedency so I'll adhere to
  built-in `Task.async_stream/5` solution for my future projects as long
  as it meets my requirements.

- [Flow]({% post_url 2018-12-19-elixir-flow %})

  each process is called a stage (implemented as `GenStage`).

  `Flow` is geared towards large-scale data processing on par with, say,
  `Apache Spark` though being not distributed per se unlike the latter.
