---
layout: post
title: Docker Compose - Tips
date: 2019-02-18 12:20:53 +0300
access: public
comments: true
categories: [docker]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

## (how to) revert container to its original image

it might be useful to discard all data persisted inside container.

stop and remove service container:

```sh
$ docker-compose stop db
$ docker-compose rm db
$ docker-compose up db
```

## (how to) rebuild image

it might be necessary when configuration options applied at build time are
changed - say, you want to edit docker image tag:

```diff
  # postgresql.dockerfile

- FROM postgres:10.4
+ FROM postgres:11.1
```

> <https://github.com/docker/compose/issues/1487#issuecomment-107048571>
>
> `docker-compose up` never rebuilds an image.
>
> You can run `docker-compose build` to build the images.

```sh
$ docker-compose stop db
$ docker-compose build db
$ docker-compose rm db
$ docker-compose up db
```

NOTE: this destroys all data - if you want to migrate data, see the next tip.

## (how to) migrate data from old Postgres version

1. <https://peter.grman.at/upgrade-postgres-9-container-to-10/>

```diff
  # docker-compose.yml

  version: '2'
  services:
    db:
-     image: postgres:9.4
+     image: postgres:12.0
```

now you need to migrate data - otherwise you'll get such error:

```
$ docker-compose up db
...
db_1  | 2019-02-18 09:19:23.315 UTC [1] DETAIL:  The data directory was
  initialized by PostgreSQL version 10, which is not compatible with this
  version 11.1 (Debian 11.1-1.pgdg90+1).
```

- backup database

  1. [PostgreSQL - Tips]({% post_url 2017-07-20-postgresql-tips %})

  ```sh
  $ docker-compose exec db pg_dumpall -U postgres > dump.sql
  ```

- rebuild image

  ```sh
  $ docker-compose stop db
  $ docker-compose build db
  $ docker-compose rm db
  $ docker-compose up db
  ```

- restore database

  1. [PostgreSQL - Tips]({% post_url 2017-07-20-postgresql-tips %})

  > <https://www.postgresql.org/docs/9.1/backup-dump.html#BACKUP-DUMP-ALL>
  >
  > Actually, you can specify any existing database name to start from...
  >
  > pg_dumpall works by emitting commands to re-create roles, tablespaces, and
  > empty databases, then invoking pg_dump for each database.

  ```sh
  $ cat dump.sql | eval 'docker-compose exec -T db psql -U "postgres" -d "myapp_development"'
  ```
