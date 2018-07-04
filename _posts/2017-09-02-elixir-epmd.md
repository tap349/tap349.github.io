---
layout: post
title: Elixir - epmd
date: 2017-09-02 13:00:52 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

epmd is like a DNS server for Erlang nodes.

<https://chazsconi.github.io/2017/04/22/observing-remote-elixir-docker-nodes.html>:

> epmd (Erlang Port Mapper Daemon), is a part of the Erlang runtime system,
> written in C, that acts as a name server for distributed Erlang. When an
> Erlang node starts in distributed mode (by setting the -name parameter on
> startup) it checks to see if there is already an EPMD instance running bound
> to the loopback address (and by default listening on port 4369), and if not
> starts one. It then chooses a random port for inter-node communication, and
> registers its name and corresponding port with EPMD.
>
> Normally each server will have its own EPMD instance, but each EPMD instance
> can have multiple Erlang nodes on that server registered to it.

<http://erlang.org/doc/reference_manual/distributed.html>:

> epmd is responsible for mapping the symbolic node names to machine addresses.

1. <http://erlang.org/doc/man/epmd.html>
2. <http://erlang.org/doc/reference_manual/distributed.html>

NOTE: Elixir application (Erlang node) can respond to application
      commands (ping/start/stop) iff its name is registered in epmd.

starting
--------

epmd can be started:

- detached (as a daemon)

  normally epmd is to be started automatically as a daemon when
  distributed Erlang node is started and no running instance of
  epmd is present (otherwise it's used).

  <http://erlang.org/doc/man/erl.html#start_epmd>:

  it's possible to instruct distributed Erlang node not to start
  epmd on startup by passing `-start_epmd false` EVM flag - in
  that case node will fail to start if epmd is not running.

  but in my experience it had no effect - application still started
  epmd unless it was already running.

- in the foreground

  epmd is usually started in the foreground either in systemd
  service unit or manually in the shell for debugging purposes.

debugging using epmd
--------------------

- kill running epmd process and start in the foreground for debugging

  ```sh
  $ sudo killall epmd
  $ epmd -d
  ```

- list names registered with currently running epmd

  ```sh
  $ epmd -names
  epmd: up and running on port 4369 with data:
  name billing_stage at port 30701
  name billing_prod at port 30183
  ```

troubleshooting
---------------

### node is unregistered right after it's registered

this might happen when application crashes - examine its _erl_crash.dump_
file or try to start application in the foreground to find out the error.

### successfully running node is not registered

situation when application is successfully running but not
registered in epmd can be caused by this sequence of events:

- application that is started via systemd service first
  (say, `billing_prod`) spawns `epmd` process
- restarting `billing_stage` service doesn't affect `epmd` process
- stopping `billing_prod` service kills `epmd` process
- starting `billing_prod` service again starts new `epmd` process
  (because there are no running instances of `epmd` any longer)

as a result new `epmd` process knows nothing about `billing_stage`
application (even though it's still running) and the latter stops
responding to all application commands.

IDK why but stopping `billing_prod` service again doesn't kill
`epmd` process any longer.

**solution**

1. <https://bugzilla.redhat.com/show_bug.cgi?id=1104843>
2. <https://stackoverflow.com/questions/17324786>

in theory `epmd` process shouldn't actually exit when application
that started it automatically is stopped. but for some reason this
is the case when application is started via systemd service.

so it seems safer to make running epmd independent of running applications -
this can be done by creating a dedicated systemd service for epmd and
configuring it as a requirement dependency for all application services:

```
Requires=epmd.service
```

quick and dirty fix to register not responding application in epmd:

the only way to make epmd aware of running but not responding
application is to restart its service - I guess systemd sends SIGTERM
signal to running process when it fails to stop it using `ExecStop=`
command. when application is started it will discover already running
epmd instance and register itself as usual.
