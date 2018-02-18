---
layout: post
title: Elixir - BEAM
date: 2018-02-19 00:12:25 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://stackoverflow.com/a/46869400/3632318>

    In principle everything is copied. Every process has its own heap.

    In reality there are a few underlying speed hacks. The most notable are:

    - Literals known at compile time are referenced from the global heap
      (which is some cases is a huge performance gain)
    - Binaries larger than 64 bytes are referenced from the global heap
      (which also causes binaries to be a leaky abstraction, hence `binary:copy/1,2`)
    - Updates to most structures do not actually require copying the whole
      structure (of particular interest is what goes on inside maps) - but
      how much and when copying is necessary is ever changing as more efficiency
      work goes into the runtime
    - Garbage collection occurs per process which is why Erlang appears to have
      a magically advanced incremental GC scheme, but actually has quite boring
      generational heap collection underneath (in the general case, that is; the
      approach is actually somewhat of a hybrid)
