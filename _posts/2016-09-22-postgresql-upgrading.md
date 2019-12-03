---
layout: post
title: PostgreSQL - Upgrading
date: 2016-09-22 11:37:26 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## 9.4 → 9.5

1. <https://www.postgresql.org/docs/9.5/static/pgupgrade.html>
2. <https://www.postgresql.org/docs/10/static/pgupgrade.html>
3. <https://www.postgresql.org/docs/10/static/upgrading.html>

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

`install` command will also initialize new database but you can always do it
manually if something goes wrong:

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

uninstall 9.4 (remove all installed versions and
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

## 10.6 → 11.1

just run `brew postgresql-upgrade-database`:

```
$ brew upgrade postgresql
...
To migrate existing data from a previous major version of PostgreSQL run:
  brew postgresql-upgrade-database
```

```
$ brew postgresql-upgrade-database
==> Upgrading postgresql data from 10 to 11...
Stopping `postgresql`... (might take a while)
==> Successfully stopped `postgresql` (label: homebrew.mxcl.postgresql)
==> Moving postgresql data from /usr/local/var/postgres to /usr/local/var/postgres.old...
The files belonging to this database system will be owned by user "tap".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /usr/local/var/postgres ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    /usr/local/opt/postgresql/bin/pg_ctl -D /usr/local/var/postgres -l logfile start

Performing Consistency Checks
-----------------------------
Checking cluster versions                                   ok
Checking database user is the install user                  ok
Checking database connection settings                       ok
Checking for prepared transactions                          ok
Checking for reg* data types in user tables                 ok
Checking for contrib/isn with bigint-passing mismatch       ok
Creating dump of global objects                             ok
Creating dump of database schemas
                                                            ok
Checking for presence of required libraries                 ok
Checking database user is the install user                  ok
Checking for prepared transactions                          ok

If pg_upgrade fails after this point, you must re-initdb the
new cluster before continuing.

Performing Upgrade
------------------
Analyzing all rows in the new cluster                       ok
Freezing all rows in the new cluster                        ok
Deleting files from new pg_xact                             ok
Copying old pg_xact to new server                           ok
Setting next transaction ID and epoch for new cluster       ok
Deleting files from new pg_multixact/offsets                ok
Copying old pg_multixact/offsets to new server              ok
Deleting files from new pg_multixact/members                ok
Copying old pg_multixact/members to new server              ok
Setting next multixact ID and offset for new cluster        ok
Resetting WAL archives                                      ok
Setting frozenxid and minmxid counters in new cluster       ok
Restoring global objects in the new cluster                 ok
Restoring database schemas in the new cluster
                                                            ok
Copying user relation files
                                                            ok
Setting next OID for new cluster                            ok
Sync data directory to disk                                 ok
Creating script to analyze new cluster                      ok
Creating script to delete old cluster                       ok

Upgrade Complete
----------------
Optimizer statistics are not transferred by pg_upgrade so,
once you start the new server, consider running:
    ./analyze_new_cluster.sh

Running this script will delete the old cluster's data files:
    ./delete_old_cluster.sh
==> Upgraded postgresql data from 10 to 11!
==> Your postgresql 10 data remains at /usr/local/var/postgres.old
==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)
```
