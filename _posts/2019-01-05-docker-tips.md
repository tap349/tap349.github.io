---
layout: post
title: Docker - Tips
date: 2019-01-05 18:52:52 +0300
access: public
comments: true
categories: [docker]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) build and publish image
--------------------------------

1. <https://docs.docker.com/get-started/part2>

```sh
$ ls
Dockerfile
$ docker build --tag=es-nginx:0.0.1 .
$ docker login
$ docker tag es-nginx:0.0.1 tap349/es-nginx:0.0.1
$ docker push tap349/es-nginx:0.0.1
```

`docker tag` creates a different tag for the same image - image ID remains
the same:

```sh
$ docker image ls
REPOSITORY       TAG    IMAGE ID      CREATED        SIZE
es-nginx         0.0.1  ff36bcadf7fe  3 minutes ago  109MB
tap349/es-nginx  0.0.1  ff36bcadf7fe  3 minutes ago  109MB
```

usually it's used to "tag image into another repository" so that it can be
automatically uploaded to [Docker Hub](https://hub.docker.com) later.
