---
layout: post
title: dry-rb - dry-initializer
date: 2017-12-09 04:02:03 +0300
access: public
comments: true
categories: [dry-rb]
---

<!-- more -->

* TOC
{:toc}
<hr>

optional attributes
-------------------

<http://dry-rb.org/gems/dry-initializer/optionals-and-defaults>:

```ruby
# `game` option can be not specified at all
option :game, Types.Instance(Game), optional: true
```
