---
layout: post
title: rails-caching
date: 2016-03-20 16:17:42 +0300
access: public
categories: [rails, caching]
---

notes about caching in rails.

<!-- more -->

* TOC
{:toc}

<https://www.nateberkopec.com/2015/07/15/the-complete-guide-to-rails-caching.html>

# general

caching:

- application-layer caching
- HTTP caching

response-time threshold for user is 1 second or less
(from the moment of click till DOM finishes painting).

but this includes:

- network latency
- loading JS and CSS resources
- building render tree
- painting
- execution of JS scripts (before and after DOM is ready)

=> server response should be below ~300ms

# profiling

applications must be profiled:

- in production mode:
  - code reloading disabled
  - assets precompiled
- with production data:
  - production DB backup
  - appropriate seed data

**MAART** - maximum acceptable average response time (e.g. 50ms)

## 3rd party tools

- rack-mini-profiler
- wrk
- apache bench

## rails logs

total time is the important one.

`Views` time might include executing queries since AR relations are lazily loaded
`ActiveRecord` time is actual time spent in DB

# caching techniques

complicated part of caching is knowing when to expire caches.

## key-based cache expiration

key-based expiration is a cache expiration strategy that expires entries
in the cache by making cache key contain information about the value
being cached -> when object changes cache key changes as well.
previous cache key is never used again and cache store expires when it, say,
runs out of memory - we never expire cache keys manually.

using `touch` option on `belongs_to` associations allows to update
associated object whenever current object is touched or saved.

cache key used in fragment caching:

`views/todos/123-20120806214154/7a1156131a6928cb0026877f8b749ac9`

which means:

`views/<object.class.name.underscore.pluralize>/<object.id>-<object.updated_at>/<md5 sum of template tree>`

changing either object itself or template tree expires cache key
(actually cache keys are not expired - they are just left unused).

when array is used as cache key for `cache` final cache key is a
concatenated version of everything in the array (with `/` as delimiter).

## russian doll caching

**russian doll caching** is simply using key-based cache expiration.

# cache digests

<https://github.com/rails/cache_digests>

**cache digests** were included in rails 4 and had been available as a gem before.

prior to cache digests it was necessary to expire fragment caches explicitly
when changing template by adding, say, version `v1` to cache key.
now cache key contains MD5 sum of template tree (template file itself and
all of its dependencies) - whenever template changes cache expires.

# cache backends
