---
layout: post
title: PostgreSQL troubleshooting
date: 2017-04-04 15:06:42 +0300
access: public
categories: [postgresql]
---

<!-- more -->

common ways to diagnose and fix the problem:

- list running services

  ```sh
  $ brew services list
  ```

- restart the service

  ```sh
  $ brew services restart postgresql@9.5
  ```

## problems with new homebrew versioning scheme

- <https://github.com/Homebrew/brew/blob/master/docs/Versions.md>

### description

running `brew upgrade` has created quite a mess for me because of a new
versioning scheme: `postgresql95` formula is replaced with `postgresql@9.5`
(this is because homebrew/core now supports multiple versions).
but this migration was not smooth and resulted in many errors, to name a few:

- `invalid value for parameter "TimeZone": "UTC"`

  should by fixed by restarting service or the whole system according to SO.

- `command not found: psql`

  `psql` must have symlink in _/usr/local/bin_ directory
  (it has been added to `PATH` in _~/.zshenv_ in my case) -
  it's gone now for some mysterious reason.

- `psql: FATAL:  database "db_name" does not exist`

  this is because running `brew upgrade` has created new data directory for
  `postgresql@9.5` (_/usr/local/var/postgresql@9.5_) while all my databases
  are stored in _/usr/local/var/postgres_.

### solution

so this is what I did to fix problems mentioned above:

- `brew untap 'homebrew/versions'` (it's deprecated now)
- `brew untap 'caskroom/versions'` (it's deprecated now)
- `brew uninstall postgresql postgresql95 postgresql@9.5` (remove everything)
- `brew install postgresql@9.5` (install latest 9.5 version)
- `brew switch postgresql@9.5 9.5.6` (switch to latest 9.5 version)
- `brew prune postgresql@9.5` (remove old 9.5 versions - if any)
- `brew link postgresql@9.5 --force` (create symlink in _/usr/local/bin_)

  it's not recommended though - it must be better to add _bin_ directory
  of specific postgresql installation to `PATH` explicitly in _~/.zshenv_.

- `cd /usr/local/var && mv postgres postgresql@9.5` (rename directory with databases)

  it might be necessary to remove existing _postgresql@9.5_ directory
  beforehand that could be created when upgrading PostgreSQL
  (but still double check it doesn't contain any databases).

- `rm /usr/local/Cellar/postgresql95` (remove symlink to `postgresql@9.5`)

  I guess it has been created for compatibility reasons.

- `gem uninstall pg && bundle` (reinstall `pg` gem)

NOTE: installing `postgresql` formula still installs the latest version
      (9.6 as of now) - all directories are named just `postgresql` accordingly.
      but its binaries are not symlinked into _usr/bin/local_ directory
      by default (now they all point to 9.5 installation) -
      if you need it run `brew link postgresql --force` manually.

## could not connect to server: Connection refused

### description

`psql -d <my_database>`:

```sh
psql: could not connect to server: Connection refused
  Is the server running locally and accepting
  connections on Unix domain socket "/tmp/.s.PGSQL.5432"?
```

rails console:

```sh
PG::ConnectionBad: could not connect to server: Connection refused
  Is the server running on host "localhost" (127.0.0.1) and accepting
  TCP/IP connections on port 5432?
could not connect to server: Connection refused
  Is the server running on host "localhost" (::1) and accepting
  TCP/IP connections on port 5432?
```

### solution

<https://stackoverflow.com/a/13573207/3632318>

the problem usually appears after hard reboot. the latter doesn't allow
PostgreSQL to exit gracefully and delete its PID files - _postmaster.pid_
in particular. so upon reboot PostgreSQL thinks it's still running and
corresponding service fails to start.

so just delete that PID file and start the service:

```sh
$ rm /usr/local/var/postgresql@9.5/postmaster.pid
$ brew services start postgresql@9.5
```

or else try to restart the service (stopping the service might
remove obsolete PID file as well - I didn't try this method though):

```sh
$ brew services restart postgresql@9.5
```
