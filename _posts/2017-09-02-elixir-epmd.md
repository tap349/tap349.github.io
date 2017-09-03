---
layout: post
title: Elixir - EPMD
date: 2017-09-02 13:00:52 +0300
access: public
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

EPMD is like a DNS server for Erlang nodes.

<https://chazsconi.github.io/2017/04/22/observing-remote-elixir-docker-nodes.html>:

> EPMD (Erlang Port Mapper Daemon), is a part of the Erlang runtime system,
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

> EPMD is responsible for mapping the symbolic node names to machine addresses.

1. <http://erlang.org/doc/man/epmd.html>
2. <http://erlang.org/doc/reference_manual/distributed.html>

Elixir application (Erlang node) can respond to application commands
(ping/start/stop) only if its name is registered in EPMD.

## starting

EPMD can be started:

- detached (as a daemon)

  normally EPMD is to be started automatically as a daemon
  when distributed Erlang node is started and no running
  instance of EPMD is present (otherwise it's used).

- in the foreground

  EPMD is usually started in the foreground either in systemd
  service unit or manually in the shell for debugging purposes.

## debugging with EPMD

- list names registered with currently running EPMD

  ```sh
  $ epmd -names
  epmd: up and running on port 4369 with data:
  name billing_stage at port 30701
  name billing_prod at port 30183
  ```

- kill running EPMD process and start in the foreground for debugging

  ```sh
  $ sudo killall epmd
  $ epmd -d
  ```

## troubleshooting

### node is unregistered right after it's registered

this might happen when application crashes - examine its _erl_crash.dump_
file or try to start application in the foreground to find out the error.

### running node is not registered

1. <https://bugzilla.redhat.com/show_bug.cgi?id=1104843>

if application is successfully running but is not registered in EPMD
this can be caused by this sequence of events:

- application that starts first via systemd service
  (say, `billing_prod`) spawns `epmd` process as well
- restarting `billing_stage` service doesn't affect `epmd` process
- stopping `billing_prod` service kills `epmd` process
- starting `billing_prod` service again starts new `epmd process`
  (because there are no running instances of `epmd` any longer)

new `epmd` process knows nothing about `billing_stage` application
even though it's still running. as a result the latter stops responding
to all application commands.
