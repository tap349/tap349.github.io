---
layout: post
title: monads in Ruby
date: 2017-02-27 00:47:33 +0300
access: public
categories: [dry-rb, ruby]
---

notes on using [dry-monads](http://dry-rb.org/gems/dry-monads/).

<!-- more -->

## definition of monad

- https://en.wikipedia.org/wiki/Monad_(functional_programming)
- http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html
- https://vimeo.com/97344498

monad is created by defining:

- type constructors (types)

  in dry-monads `Either` monad is defined using 2 types:
  `Either::Right` and `Either::Left` (they are subclasses of `Either` class -
  but it's just a Ruby-specific implementation detail).

  to lift a value is to create a monad from a plain value: `Either::Right(5)`.
  lifted value is called a monadic value.

- operations (only the first is obligatory for all monads):

  - `bind: m a -> (a -> m b) -> m b`

    `bind` function is often aliased as `>>=` (like in Haskell).

    it takes one monad (wrapping `a`) and a function transforming `a`
    into another monad (wrapping `b`) and returns that another monad.

    in other words: it allows for a function accepting plain value to have
    monadic value as input - next `bind` function either unlifts monadic value
    (if monadic value is of one type - `Either::Right` for `Either` monad)
    feeding plain value to the function or immediately returns original monadic
    value (if it's of another type - `Either::Left` for `Either` monad).

  - `fmap: m a -> (a -> b) -> m b`

    it's used for single-track function (term from ROP) -
    they are supposed to succeed all the time.
    it just lifts the result for you (wraps return value in monad).

  - `tee: m a -> (a -> m b) -> m a` (for dry-monads only)

    in case of dry-monads `tee` function is supposed to take monadic value
    (`Try` monad doesn't implement it - only `Maybe` and `Either` monads do)
    and return monadic value as well - even though return value is discarded
    since input monadic value is always returned.

## monads in dry-monads

3 monads are available in dry-monads: `Maybe`, `Either` and `Try`.
each one has 2 type constructors:

- `Maybe`: `Some`/`None`
- `Either`: `Right`/`Left`
- `Try`: `Success`/`Failure`

note that you cannot create `Try` monadic value (`Success` or `Failure`)
explicitly - it's implicitly returned by wrapping some code into `Try` monad.
