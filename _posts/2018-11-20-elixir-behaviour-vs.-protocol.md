---
layout: post
title: Elixir - Behaviour vs. Protocol
date: 2018-11-20 22:40:46 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://groups.google.com/forum/#!msg/elixir-lang-talk/S0NlOoc4ThM/J2aD2hKrtuoJ>

> José Valim
>
> A behaviour is a way to say: give me a module as argument and I will invoke
> the following callbacks on it.

> José Valim
>
> Protocol is a behaviour with the dispatching logic so you don't need to hand
> roll it nor impose a particular implementation in the user module.

> Saša Jurić
>
> > But for practical purposes I don't see that a behaviour is any different
> > to a protocol, other than a behaviour *forces* the glue logic into the
> > datastructure module, rather than allowing it to live outside that module,
> > as with a protocol?
>
> I'm not even sure that behaviour qualifies as polymorphism, because it's
> not a data driven dispatch. You use the generic behaviour (e.g. GenServer),
> and make it concrete by providing your callback module - a plugin that is
> essentially a bunch of functions. The generic logic then drives the process
> and invokes the callback module when some concrete decisions need to be made.
>
> Since behaviour is type agnostic, we can get some flexible properties. For
> example in GenServer:
>
> - the same type (struct) can be used as the state of different concrete
>   GenServers
> - a single GenServer can easily mutate its state into some other type at
>   any point in time
>
> This can surely be implemented with protocols as well, but I think the result
> would require more boilerplate on behalf of the client programmer.
>
> So in my mind, a behaviour is more appropriate when the generic logic
> doesn't care about the data at all. It may take some data from the concrete
> implementation and return it later to that same implementation (which is
> exactly what GenServer does), but it doesn't perform any operations itself
> with that data.
>
> In my opinion, if you want to vary Heap implementations, I think protocols
> are the initial way to go here, since you're doing a data based polymorphism.

protocol is `a behaviour with dispatching logic`, `data based polymorphism`.

behaviour is more appropriate when `generic logic doesn't care about the data`
and `doesn't perform any operations with that data` but `invokes the callback
module when some concrete decisions need to be made`.
