---
layout: post
title: Phoenix - Style Guide
date: 2018-03-26 13:36:31 +0300
access: public
comments: true
categories: [phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

naming
------

### attrs vs. params

`params` is what comes from controller - they might map to `attrs` exactly or
not => use `params` everywhere except for cases when it's clear that you deal
with attributes - say, when you create a map of attributes by yourself or in
changeset functions inside schema modules.

NOTE: in Phoenix and Ecto docs they now use `params` everywhere.
