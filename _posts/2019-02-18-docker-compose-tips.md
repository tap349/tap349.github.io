---
layout: post
title: Docker Compose - Tips
date: 2019-02-18 12:20:53 +0300
access: public
comments: true
categories: [docker]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) revert container to its original image
-----------------------------------------------

it might be useful to discard all data persisted inside container.

stop and remove service container:

```sh
$ docker-compose stop db
$ docker-compose rm db
$ docker-compose up db
```

(how to) rebuild image
----------------------

it might necessary when configuration options applied at build time are
changed - say, you want to edit docker image tag:

```yaml
# docker-compose.yml

version: '2.2'
services:
  db:
    build:
      context: .
      dockerfile: postgresql.dockerfile
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - 5434:5432
```

```diff
  # postgresql.dockerfile

- FROM postgres:10.4
+ FROM postgres:11.1
```

in case of `db` service it might be necessary (1) to rebuild image AND (2) to
create a new container because in old one the data directory might have been
initialized by older version of PostgreSQL:

```
db_1  | 2019-02-18 09:19:23.315 UTC [1] DETAIL:  The data directory was
  initialized by PostgreSQL version 10, which is not compatible with this
  version 11.1 (Debian 11.1-1.pgdg90+1).
```

```sh
$ docker-compose stop db
$ docker-compose build db
$ docker-compose rm db
$ docker-compose up db
```
