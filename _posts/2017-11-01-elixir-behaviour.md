---
layout: post
title: Elixir - Behaviour
date: 2017-11-01 10:10:45 +0300
access: private
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://hexdocs.pm/elixir/behaviours.html>
2. <https://hexdocs.pm/elixir/Application.html>

3 ways to extract behaviours:

- separate module

  behaviour is extracted into separate module - it's referenced wherever
  behaviour must be implemented.

  still specific implementation module is fetched from environment at
  runtime using `Application.get_env/3` in all places where it's used.

- separate module with `defdelegate`

  same as previous one but this time this separate module is the only
  place where specific implementation is fetched - module delegates
  all public functions to fetched implementation explicitly.

  all other modules use this module's functions and are not even aware
  that it's using some implementation fetched at runtime.

  # it's possible to inject shikimori client implementations
  # (adapters) into the modules where client is used -
  # this is just another level of indirection in case you need
  # to prepare data for API call or provide some defaults

- implementation module with nested behaviour module

  it's useful when there is only one real and one mock implementation and
  when you don't want to think of how to name real implementation.

  real implementation module has nested behaviour module (usually named
  just `Behaviour`) and implements this behaviour as well.
