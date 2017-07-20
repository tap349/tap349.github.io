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

- <https://stackoverflow.com/questions/1566717>

- create B-tree index

  only left-anchored patterns (no leading wildcard) are supported by
  B-tree indexes => this index is good for auto-completion.

  ```sql
  SELECT * FROM tbl WHERE col LIKE 'foo%';
  ```

  in Rails:

  ```ruby
  add_index(:teams, :name, using: :btree)
  ```

- create trigram index

  - <https://www.postgresql.org/docs/9.6/static/pgtrgm.html>
  - <http://blog.scoutapp.com/articles/2016/07/12/how-to-make-text-searches-in-postgresql-faster-with-trigram-similarity>

  all kinds of patterns (not only left-anchored ones) are supported by
  trigram indexes => this index is good for general searches.

  > PostgreSQL splits a string into words and determines trigrams for each
  > word separately. It also normalizes the word by downcasing it, prefixing
  > two spaces and suffixing one. Non-alphanumeric characters are considered
  > to be word boundaries. Note that downcasing makes trigrams case-insensitive.

  > We don't need any special changes to the query to use the trigram index.
  > Any query using LIKE will improve.

  ```sql
  CREATE INDEX teams_on_name_idx ON teams USING GIN (name gin_trgm_ops);
  ```

  in Rails (fallback to raw SQL - Rails doesn't accept custom operator classes):

  ```ruby
  enable_extension :pg_trgm
  execute <<~SQL
    CREATE INDEX teams_on_name_idx ON teams USING GIN (name gin_trgm_ops);
  SQL
  ```
