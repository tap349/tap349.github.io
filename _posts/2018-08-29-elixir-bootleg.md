---
layout: post
title: Elixir - Bootleg
date: 2018-08-29 12:06:25 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

### release version

by default project version from _mix.exs_ is used - or else it can
be overriden in Bootleg config:

```elixir
# config/deploy.exs

config :version, "0.0.1"
```

NOTE: version is used in both `build` and `deploy` tasks!

=> `deploy` task uses version to determine which release to deploy.

### rollback release

set previous release version:

```diff
  # config/deploy.exs

- config :version, "0.0.2"
+ config :version, "0.0.1"
```

deploy release:

```sh
$ mix bootleg.deploy
```
