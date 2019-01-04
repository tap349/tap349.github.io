---
layout: post
title: Docker - Notes
date: 2019-01-04 15:21:08 +0300
access: public
comments: true
categories: [docker]
---

<!-- more -->

* TOC
{:toc}
<hr>

containers
----------

1. <https://docs.docker.com/get-started/part1>
2. <https://docs.docker.com/get-started/part2>

> <https://docs.docker.com/get-started/#images-and-containers>
>
> A container is launched by running an image.
>
> A container is a runtime instance of an image - what the image becomes in
> memory when executed (that is, an image with state, or a user process).

services
--------

1. <https://docs.docker.com/get-started/part3>

> <https://docs.docker.com/get-started/part3/#recap-and-cheat-sheet-optional>
>
> To recap, while typing docker run is simple enough, the true implementation
> of a container in production is running it as a service. Services codify a
> container’s behavior in a Compose file, and this file can be used to scale,
> limit, and redeploy our app.

NOTE: it's not necessary to initialize a swarm to use `docker-compose`.

> <https://docs.docker.com/get-started/part3/#about-services>
>
> Services are really just “containers in production.” A service only runs
> one image, but it codifies the way that image runs—what ports it should
> use, how many replicas of the container should run so the service has the
> capacity it needs, and so on.

> <https://docs.docker.com/get-started/part3/#run-your-new-load-balanced-app>
>
> A single container running in a service is called a task. Tasks are given
> unique IDs that numerically increment, up to the number of replicas you
> defined in docker-compose.yml.
