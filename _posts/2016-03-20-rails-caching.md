---
layout: post
title: rails-caching
date: 2016-03-20 16:17:42 +0300
access: private
categories: [rails, caching]
---

<https://github.com/rails/cache_digests>:

cache digest were included in rails 4 and had been available as a gem before.

prior to cache digests it was necessary to expire fragment caches explicitly
when changing template by adding, say, version `v1` to cache key.
now cache key contains md5 sum of template tree (template file itself and
all of its dependencies) - whenever template changes cache expires.

<https://www.nateberkopec.com/2015/07/15/the-complete-guide-to-rails-caching.html>:

russian doll caching is simply using key-based cache expiration.

using `touch` option on `belongs_to` associations allows to update
this association whenever current object object is touched or saved.
