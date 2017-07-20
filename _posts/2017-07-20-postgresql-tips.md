---
layout: post
title: PostgreSQL tips
date: 2017-07-20 16:09:32 +0300
access: public
categories: [postgresql, rails]
---

<!-- more -->

## psql tips

- <https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS>

### tables

- describe table

  ```sql
  \d users
  ```

### extensions

on macOS extensions are located in
_/usr/local/Cellar/postgresql/9.6.3/share/postgresql/extension/_.

- list installed extensions

  ```sql
  \dx
  SELECT * FROM pg_extension;
  ```

- list all available extensions

  ```sql
  SELECT * FROM pg_available_extensions;
  ```

- install extension

  ```sh
  $ psql -d myapp_development
  > CREATE EXTENSION pg_trgm;
  ```

## query optimization

- <https://robots.thoughtbot.com/why-postgres-wont-always-use-an-index>

### LIKE operator

- <https://stackoverflow.com/questions/1566717>

- create B-tree index

  only left-anchored patterns (no leading wildcard) are supported by
  B-tree indexes => they are good for auto-completion.

  ```sql
  SELECT * FROM tbl WHERE col LIKE 'foo%';
  ```

  in Rails migration:

  ```ruby
  add_index(:users, :name, using: :btree)
  ```

- create trigram index

  - <https://www.postgresql.org/docs/9.6/static/pgtrgm.html>
  - (!) <http://blog.scoutapp.com/articles/2016/07/12/how-to-make-text-searches-in-postgresql-faster-with-trigram-similarity>

  all kinds of patterns (not only left-anchored ones) are supported by
  trigram indexes => they are good for general searches.

  > PostgreSQL splits a string into words and determines trigrams for each
  > word separately. It also normalizes the word by downcasing it, prefixing
  > two spaces and suffixing one. Non-alphanumeric characters are considered
  > to be word boundaries. Note that downcasing makes trigrams case-insensitive.

  > We don't need any special changes to the query to use the trigram index.
  > Any query using LIKE will improve.

  ```sql
  CREATE EXTENSION pg_trgm;
  CREATE INDEX users_on_name_idx ON users USING GIN (name gin_trgm_ops);
  ```

  in Rails migration (fallback to raw SQL - Rails doesn't accept custom operator classes):

  ```ruby
  enable_extension :pg_trgm
  execute <<~SQL
    CREATE INDEX users_on_name_idx ON users USING GIN (name gin_trgm_ops);
  SQL
  ```
