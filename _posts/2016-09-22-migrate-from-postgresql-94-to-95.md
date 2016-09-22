---
layout: post
title: migrate from postgresql 9.4 to 9.5
date: 2016-09-22 11:37:26 +0300
access: public
categories: [postgresql]
---

guide on how to migrate data when switching from 9.4 to 9.5.

<!-- more -->

backup 9.4 data:

```sh
$ mv /usr/local/var/postgres /usr/local/var/postgresql94.backup
```

install 9.5 (the last step is optional most likely):

```sh
$ brew services stop postgresql94
$ brew unlink postgresql94
$ brew install postgresql
$ brew link postgresql
```

this will also initialize new database but you can always do it manually:

```sh
$ initdb /usr/local/var/postgres
```

make sure you are using pg_upgrade from 9.5:

```sh
$ pg_upgrade --version
pg_upgrade (PostgreSQL) 9.5.4
```

migrate 9.4 data:

```sh
$ pg_upgrade \
  -d /usr/local/var/postgres94.backup \
  -D /usr/local/var/postgres \
  -b /usr/local/Cellar/postgresql94/9.4.9/bin \
  -B /usr/local/Cellar/postgresql/9.5.4/bin
```

uninstall 9.4 (and remove any 9.4 versions manually if any left):

```sh
$ brew uninstall postgresql94
$ rm -rf /usr/local/Cellar/postgresql94
```

remove 9.5 data (optionally):

```sh
$ rm -rf /usr/local/var/postgresql94.backup
```

start 9.5 service:

```sh
$ brew services start postgresql
```
