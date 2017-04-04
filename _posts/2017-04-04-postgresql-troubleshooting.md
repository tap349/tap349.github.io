---
layout: post
title: postgresql troubleshooting
date: 2017-04-04 15:06:42 +0300
access: public
categories: [postgresql]
---

<!-- more -->

### use new homebrew versioning scheme

- <https://github.com/Homebrew/brew/blob/master/docs/Versions.md>

running `brew upgrade` has created quite a mess for me because of a new
versioning scheme: `postgresql95` formula is replaced with `postgresql@9.5`
(this is because homebrew/core now supports multiple versions).
but this migration was not smooth and resulted in many errors, to name a few:

- `invalid value for parameter "TimeZone": "UTC"`

  should by fixed by restarting service or the whole system according to SO.

- `command not found: psql`

  `psql` must have symlink in _/usr/local/bin_ directory
  (it has been added to $PATH in _~/.zshenv_ in my case) -
  it's gone now for some mysterious reason.

- `psql: FATAL:  database "db_name" does not exist`

  this is because running `brew upgrade` has created new data directory for
  `postgresql@9.5` (_/usr/local/var/postgresql@9.5_) while all my databases
  are stored in _/usr/local/var/postgres_.

so this is what I did to fix problems mentioned above:

- `brew untap 'homebrew/versions'` (since it's deprecated)
- `brew untap 'caskroom/versions'` (since it's deprecated)
- `brew uninstall postgresql postgresql@9.5`
- `brew install postgresql@9.5` (install latest 9.5 version)
- `brew switch postgresql@9.5 9.5.6` (switch to latest 9.5 version)
- `brew prune postgresql@9.5` (remove old 9.5 versions - if any)
- `brew link postgresql@9.5 --force` (create symlink in _/usr/local/bin_)

  though it's not recommended - I guess it's better to add _bin_ directory
  of specific postgresql installation to `PATH` explicitly in _~/.zshenv_.

- `cd /usr/local/var && mv postgres postgresql@9.5`
  (rename directory with databases to new format)

  it might be necessary to remove existing _postgresql@9.5_ directory
  that was created when upgrading postgresql to use new version format
  (but still double check it doesn't contain any databases).

- `rm /usr/local/Cellar/postgresql95` (symlink to `postgresql@9.5`)
