---
layout: post
title: PostgreSQL - Tuning
date: 2019-04-18 11:45:45 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server>

max_connections
---------------

1. <https://wiki.postgresql.org/wiki/Number_Of_Database_Connections>

the maximum number of client connections (sessions) allowed is set by
`max_connections` setting in _postgresql.conf_ - its value defaults to 100.

show current value of `max_connections` setting in `psql`:

```sql
SHOW max_connections;
```

### max_connections vs. pool_size of Ecto repository

each Ecto repository has a `pool_size` option - the number of connections to
keep in the pool. say, my umbrella application has 3 child applications each
one having repository with a pool size 15 => so there are always 15 * 3 = 45
open connections and there remains 100 - 45 = 55 available connection slots
(provided `max_connections` setting has default value of 100).

### increase max_connections

say, to increase `max_connections` from 100 to 200:

```diff
  # /etc/postgresql/11/main/postgresql.conf

- max_connections = 100
+ max_connections = 200
```

it's necessary to restart PostgreSQL service for the changes to take effect:

```sh
$ sudo systemctl restart postgresql
```
