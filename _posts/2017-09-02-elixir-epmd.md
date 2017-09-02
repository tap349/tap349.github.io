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

TL;DR: EPMD is like DNS server for Erlang nodes.

<https://chazsconi.github.io/2017/04/22/observing-remote-elixir-docker-nodes.html>:

> EPMD (Erlang Port Mapper Daemon), is a part of the Erlang runtime system,
> written in C, that acts as a name server for distributed Erlang. When an
> Erlang node starts in distributed mode (by setting the -name parameter on
> startup) it checks to see if there is already an EPMD instance running bound
> to the loopback address (and by default listening on port 4369), and if not
> starts one. It then chooses a random port for inter-node communication, and
> registers its name and corresponding port with EPMD.

