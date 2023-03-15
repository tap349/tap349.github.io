---
layout: post
title: Clojure debugging
date: 2023-03-15 17:15:08 +0200
access: public
comments: true
categories: []
---

<!-- more -->

* TOC
{:toc}
<hr>

## Print result in threading macros

```clojure
(→ "foo"
   (doto println)
   do-something-else)
```
