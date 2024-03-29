---
layout: post
title: Elixir - Registry
date: 2017-10-29 21:44:12 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

it's common to have a registry of processes where each process can be
accessed by its name (say, registry of user stores accessed by user_id).

registry can be implemented in several ways based on
[how](https://hexdocs.pm/elixir/GenServer.html#module-name-registration)
new process is going to be registered in that registry:

### using atom

- GenServer with manual registration and monitoring of tracked processes

  this type of registry is described in offical Elixir's getting started guide:

  - each process is started using `start_child/2` of its supervisor
  - each process is monitored using `Process.monitor/1`
  - mapping process names to their pids can be done in:
    - a simple map which is stored as part of registry state
    - ETS table (then ETS table name is part of registry state)

### using `{:via, module, term}` tuple

from now on I'll refer to registries accessed with via tuple as via registries.

the benefit of via registries is that you don't have to interact with
them directly when registering new processes - it's done automatically
when using via tuple in process client API (each process is GenServer
itself as a rule).

via registry is started as part of supervision tree by top-level supervisor.

- GenServer exporting `register_name/2`, `unregister_name/1`, `whereis_name/1` and `send/2`

  1. <https://m.alphasights.com/process-registry-in-elixir-a-practical-example-4500ee7c0dcc>

  any via registry must implement these functions
  (but there is no corresponding behaviour for some reason).

- `Registry` module introduced in Elixir 1.4

  1. <https://hexdocs.pm/elixir/Registry.html>

  it's just a standard (official) implementation of via registry.

  typically this registry is started by top-level supervisor through:

  ```elixir
  {Registry, keys: :unique, name: Registry.ViaTest}
  ```
