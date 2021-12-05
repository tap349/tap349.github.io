---
layout: post
title: PostgreSQL - Troubleshooting
date: 2017-04-04 15:06:42 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

## problems with new homebrew versioning scheme

1. <https://github.com/Homebrew/brew/blob/master/docs/Versions.md>

running `brew upgrade` has created quite a mess for me because of a new
versioning scheme: `postgresql95` formula is replaced with `postgresql@9.5`
(this is because homebrew/core now supports multiple versions). but this
migration was not smooth and resulted in many errors, to name a few:

- `invalid value for parameter "TimeZone": "UTC"`

  should be fixed by restarting service or the whole system according to SO.

- `command not found: psql`

  `psql` must have a symlink in _/usr/local/bin/_ (it has been added to `PATH`
  in _~/.zshenv_ in my case) - it's gone now for some mysterious reason.

- `psql: FATAL: database "db_name" does not exist`

  this is because running `brew upgrade` has created new data directory for
  `postgresql@9.5` (_/usr/local/var/postgresql@9.5_) while all my databases are
  stored in _/usr/local/var/postgres_.

**solution**

so this is what I did to fix problems mentioned above:

- `brew untap 'homebrew/versions'` (it's deprecated now)
- `brew untap 'caskroom/versions'` (it's deprecated now)
- `brew uninstall postgresql postgresql95 postgresql@9.5` (remove everything)
- `brew install postgresql@9.5` (install latest 9.5 version)
- `brew switch postgresql@9.5 9.5.6` (switch to latest 9.5 version)
- `brew prune postgresql@9.5` (remove old 9.5 versions - if any)
- `brew link postgresql@9.5 --force` (create symlink in _/usr/local/bin_)

  it's not recommended though - it must be better to add _bin_ directory of
  specific PostgreSQL installation to `PATH` explicitly in _~/.zshenv_.

- `cd /usr/local/var && mv postgres postgresql@9.5` (rename directory with
  databases)

  it might be necessary to remove existing _postgresql@9.5_ directory beforehand
  that could be created when upgrading PostgreSQL (but still double check it
  doesn't contain any databases).

- `rm /usr/local/Cellar/postgresql95` (remove symlink to `postgresql@9.5`)

  I guess it has been created for compatibility reasons.

- `gem uninstall pg && bundle` (reinstall `pg` gem)

NOTE: installing `postgresql` formula still installs the latest version (9.6 as
of now) - all directories are named just `postgresql` accordingly. but its
binaries are not symlinked into _usr/bin/local_ directory by default (now they
all point to 9.5 installation) - if you need it run
`brew link postgresql --force` manually.

## psql: could not connect to server: Connection refused

```
$ psql -d <DB_NAME>
psql: could not connect to server: Connection refused
  Is the server running locally and accepting
  connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

rails console:

```
PG::ConnectionBad: could not connect to server: Connection refused
  Is the server running on host "localhost" (127.0.0.1) and accepting
  TCP/IP connections on port 5432?
could not connect to server: Connection refused
  Is the server running on host "localhost" (::1) and accepting
  TCP/IP connections on port 5432?
```

**solution**

<https://stackoverflow.com/a/13573207/3632318>

the problem usually appears after hard reboot. the latter doesn't allow
PostgreSQL to exit gracefully and delete its PID files - _postmaster.pid_ in
particular. so upon reboot PostgreSQL thinks it's still running and
corresponding service fails to start.

so just delete that PID file and start the service:

```sh
$ rm /usr/local/var/postgresql@9.5/postmaster.pid
$ brew services start postgresql@9.5
```

or else try to restart the service (stopping the service might remove obsolete
PID file - I didn't try this method though):

```sh
$ brew services restart postgresql@9.5
```

## command not found: psql

```
$ psql --version
zsh: command not found: psql
```

**solution**

in my case only versioned formula of PostgreSQL (`postgresql@9.5`) was installed
but it didn't create a symlink to `psql` in _/usr/local/bin/_.

to solve this problem either:

- install the latest version of PostgreSQL (`postgresql`)

  unversioned formula creates a symlink in _/usr/local/bin/_ automatically.

- create symlinks for binaries from old version of PostgreSQL manually

  ```sh
  $ ln -s /usr/local/Cellar/postgresql@9.5/9.5.10/bin/psql /usr/local/bin/psql
  ```

- add the whole _bin/_ from old version of PostgreSQL to `PATH`

  _~/.zshenv_:

  ```zsh
  path=(/usr/local/Cellar/postgresql@9.5/9.5.10/bin $path)
  ```

## psql: could not connect to server: No such file or director

```
$ psql -d postgres
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

**solution**

to diagnose this and similar problems run `postgres` in the foreground (see
`brew info postgresql output`):

```
$ pg_ctl -D /usr/local/var/postgres start
waiting for server to start....
[10867] FATAL: database files are incompatible with server
[10867] DETAIL: The data directory was initialized by PostgreSQL version 9.6, which is not compatible with this version 10.3.
 stopped waiting
pg_ctl: could not start server
Examine the log output.
```

it turns out _/usr/local/var/postgres_ contains data for PostgreSQL 9.6 (when
`postgresql` formula was installed, 9.6 was the latest version).

=> it's necessary to migrate existing data from a previous major version (9.6)
to the latest one (10) (see `brew info postgresql output`):

```
$ brew postgresql-upgrade-database
==> brew install postgresql@9.6
...
==> Upgrading postgresql data from 9.6 to 10...
Stopping `postgresql`... (might take a while)
==> Successfully stopped `postgresql` (label: homebrew.mxcl.postgresql)
==> Moving postgresql data from /usr/local/var/postgres to /usr/local/var/postgres.old...
The files belonging to this database system will be owned by user "tap".
This user must also own the server process.
...
==> Upgraded postgresql data from 9.6 to 10!
==> Your postgresql 9.6 data remains at /usr/local/var/postgres.old
==> Successfully started `postgresql` (label: homebrew.mxcl.postgresql)
```

this command will install a previous major version of PostgreSQL (if it has been
uninstalled) which is required to upgrade database.

after upgrade is complete, you can remove old major version along with its data
(_/usr/local/var/postgres.old_):

```sh
$ brew uninstall postgresql@9.6
$ rm -rf /usr/local/var/postgres.old
```

now make sure `postgresql` service is started:

```
$ brew services list
Name       Status  User Plist
memcached  started tap  /Users/tap/Library/LaunchAgents/homebrew.mxcl.memcached.plist
postgresql started tap  /Users/tap/Library/LaunchAgents/homebrew.mxcl.postgresql.plist
redis      started tap  /Users/tap/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

and try to run `psql` again:

```
$ psql -d postgres
psql (10.3)
Type "help" for help.

postgres=#
```

## Error: Invalid data directory for cluster 10 main

```sh
$ psql -U sith_production -d sith_production
Error: Invalid data directory for cluster 10 main
```

**solution**

1. <https://stackoverflow.com/a/26183931/3632318>

add `-h localhost` option:

```sh
$ psql -U sith_production -d sith_production -h localhost
/ enter password for database user sith_production
```

for this to be possible it's necessary to have this line in
_/etc/postgresql/10/main/pg_hba.conf_:

```conf
local   sith_production sith_production                         md5
```

restart `postgresql` service for changes to take effect.

<https://stackoverflow.com/a/26735105/3632318>:

> Authentication methods details: trust - anyone who can connect to the server
> is authorized to access the database peer - use client's operating system user
> name as database user name to access it. md5 - password-base authentication

## FATAL: remaining connection slots are reserved for non-replication superuser connections

this error indicates you have run out of connections.

**solution**

see [PostgreSQL - Tuning]({% post_url 2019-04-18-postgresql-tuning %}) on how to
increase the maximum number of allowed connections.

see [PostgreSQL - Monitoring]({% post_url 2019-04-18-postgresql-monitoring %})
on how to monitor open connections.

## FATAL: sorry, too many clients already

most likely this error is also caused by a low number of allowed connections -
see solution for the error above.

## could not access the server configuration file "/etc/postgresql/12/main/postgresql.conf"

```
$ sudo -u postgres /usr/lib/postgresql/12/bin/postgres \
  -D /var/lib/postgresql/12/main \
  -c config_file=/etc/postgresql/12/main/postgresql.conf

postgres: could not access the server configuration file "/etc/postgresql/12/main/postgresql.conf": No such file or directory
```

**solution**

[1]: {% post_url 2017-07-20-postgresql-tips %}

the problem was with already existing cluster.

first I reinstalled PostgreSQL manually:

```sh
$ sudo apt autoremove --purge postgresql-12
$ sudo rm -rf /etc/postgresql
$ sudo rm -rf /run/postgresql
$ sudo apt install postgresql-12
```

but cluster from previous installation must have remained in the system so a new
cluster was not created when running the last command:

```
$ sudo apt install postgresql-12
...
Configuring already existing cluster (configuration: /etc/postgresql/12/main, data: /var/lib/postgresql/12/main, owner: 113:118)
Error: move_conffile: required configuration file /var/lib/postgresql/12/main/postgresql.conf does not exist
Error: could not create default cluster. Please create it manually with

  pg_createcluster 12 main --start

or a similar command (see 'man pg_createcluster').
```

=> solution is to remove old clusters manually and run installation again:

```sh
$ sudo rm -rf /var/lib/postgresql
$ sudo apt install postgresql-12
```

see the tip on how to remove PostgreSQL completely in [PostgreSQL - Tips][1].
