---
layout: post
title: PostgreSQL - Debugging
date: 2019-11-15 15:05:10 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://serverfault.com/a/587275>

sometimes PostgreSQL server might fail to start and there are no errors in
systemd journal:

```sh
$ sudo journalctl -u postgresql
```

or else PostgreSQL service status might be `active` but you still cannot to
server:

```sh
$ sudo systemctl status postgresql
```

=> try to run PostgreSQL server in the foreground to get actual errors:

```sh
$ sudo -u postgres /usr/lib/postgresql/12/bin/postgres \
  -D /var/lib/postgresql/12/main \
  -c config_file=/etc/postgresql/12/main/postgresql.conf
```

see the output of `dpkg-query -L postgresql-12` command (shows the contents of
package) for location of `postgres` executable.

see _/etc/postgresql/12/main/postgresql.conf_ (`data_directory` configuration
setting) for location of database directory (`-D` option).
