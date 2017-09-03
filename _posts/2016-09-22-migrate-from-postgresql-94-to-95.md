---
layout: post
title: migrate from PostgreSQL 9.4 to 9.5
date: 2016-09-22 11:37:26 +0300
access: public
categories: [postgresql]
---

guide on how to migrate data when switching from 9.4 to 9.5.

<!-- more -->

1. <https://www.postgresql.org/docs/9.5/static/pgupgrade.html>

backup 9.4 data:

```sh
$ mv /usr/local/var/postgres /usr/local/var/postgresql94.backup
```

install 9.5 (the last step is optional):

```sh
$ brew services stop postgresql94
$ brew unlink postgresql94
$ brew install postgresql
$ brew link postgresql
```

install command will also initialize new database but
you can always do it manually if something goes wrong:

```sh
$ initdb /usr/local/var/postgres
```

make sure you are using `pg_upgrade` from 9.5:

```sh
$ pg_upgrade --version
pg_upgrade (PostgreSQL) 9.5.4
```

migrate 9.4 data:

```sh
$ pg_upgrade \
  -d /usr/local/var/postgresql94.backup \
  -D /usr/local/var/postgres \
  -b /usr/local/Cellar/postgresql94/9.4.9/bin \
  -B /usr/local/Cellar/postgresql/9.5.4/bin
```

the same in one line (for copy-paste):

```sh
$ pg_upgrade -d /usr/local/var/postgresql94.backup -D /usr/local/var/postgres -b /usr/local/Cellar/postgresql94/9.4.9/bin -B /usr/local/Cellar/postgresql/9.5.4/bin
```

uninstall 9.4 (remove all versions and
_/usr/local/Cellar/postgresql94/_ directory itself):

```sh
$ brew uninstall --force postgresql94
```

remove 9.4 data (optionally):

```sh
$ rm -rf /usr/local/var/postgresql94.backup
```

start 9.5 service:

```sh
$ brew services start postgresql
```

P.S. also 2 scripts are generated after `pg_upgrade` in PWD:

- _analyze_new_cluster.sh_ (generate statistics for 9.5)
- _delete_old_cluster.sh_ (delete 9.4 data - same command as above)
