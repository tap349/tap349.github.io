---
layout: post
title: postgresql troubleshooting
date: 2017-04-04 15:06:42 +0300
access: public
categories: [postgresql]
---

<!-- more -->

### `psql not found` after upgrading postgresql with brew

`psql` must have symlink in _/usr/local/bin_ directory
(it has been added to `PATH` in _~/.zshenv_ by myself).

but symlink doesn't exist after `brew upgrade` for some reason:

- `brew switch postgresql@9.5 9.5.6` to switch to latest 9.5 version
- `brew link postgresql@9.5 --force` to create link in _/usr/local/bin_
