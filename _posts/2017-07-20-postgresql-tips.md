---
layout: post
title: PostgreSQL tips
date: 2017-07-20 16:09:32 +0300
access: public
categories: [postgresql]
---

<!-- more -->

## extensions

on macOS extensions are located in
_/usr/local/Cellar/postgresql/9.6.3/share/postgresql/extension/_.

### list installed extensions:

```
> \dx
> SELECT * FROM pg_extension;
```

### list all available extensions:

```
> SELECT * FROM pg_available_extensions;
```

### install extension

```sh
$ psql -d myapp_development
> CREATE EXTENSION pg_trgm;
```

## query optimization

### LIKE operator

<https://stackoverflow.com/questions/1566717>

- prefer left-anchored patterns (no leading wildcard)

  ```sql
  SELECT * FROM tbl WHERE col LIKE 'foo%';
  ```

- create trigram index on column string column

  - <https://www.postgresql.org/docs/9.6/static/pgtrgm.html>
  - <http://blog.scoutapp.com/articles/2016/07/12/how-to-make-text-searches-in-postgresql-faster-with-trigram-similarity>

  > not left-anchored patterns are not supported by B-tree indexes

  > downcasing makes trigrams case-insensitive

  ```sql
  CREATE INDEX teams_on_name_idx ON teams USING GIN (name gin_trgm_ops);
  ```
