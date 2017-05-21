---
layout: post
title: elixir tips
date: 2017-05-21 12:08:46 +0300
access: public
categories: [elixir]
---

<!-- more -->

### mix tasks

- `mix deps.get` = `bundle install`

  installs new dependencies and updates existing ones up to the version
  specified in _mix.lock_

- `mix deps.clean --unused --unlock` = `bundle install`

  removes dependencies which are no longer mentioned in _mix.exs_
  (`--unused`) and updates _mix.lock_ (`--unlock`)

