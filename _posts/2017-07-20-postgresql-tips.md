---
layout: post
title: PostgreSQL - Tips
date: 2017-07-20 16:09:32 +0300
access: public
categories: [postgresql, rails]
---

<!-- more -->

* TOC
{:toc}

<hr>

## psql tips

- <https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS>

### users

- list roles (= users) and their attributes

  ```
  \u
  ```

### databases

- list databases and their owners

  ```
  \l
  ```

### tables

- show all tables

  ```
  \d
  ```

  this commands also lists sequences. to show tables only:

  ```
  \dt
  ```

- show table

  ```
  \d users
  ```

### extensions

on macOS extensions are located in
_/usr/local/Cellar/postgresql/9.6.3/share/postgresql/extension/_.

- list installed extensions

  ```
  \dx
  ```

  or

  ```sql
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

<https://robots.thoughtbot.com/why-postgres-wont-always-use-an-index>

### Rails notes

- custom index operator classes

  Rails doesn't allow to specify custom operator classes when creating index
  in migrations - fallback to raw SQL in migrations in such cases:

  ```ruby
  execute <<~SQL
    CREATE INDEX users_on_name_idx ON users USING GIN (name gin_trgm_ops);
  SQL
  ```

- using extensions

  there is no need to enable extension in migration itself using
  `enable_extension :pg_trgm` statement if it has already been created
  manually in psql - `enable_extension "btree_gin"` line will be added
  to _schema.rb_ anyway after running migration.

### LIKE operator

<https://stackoverflow.com/questions/1566717>

- create B-tree index

  only left-anchored patterns (no leading wildcard) are supported by
  B-tree indexes => they are good for auto-completion.

  ```sql
  SELECT * FROM tbl WHERE col LIKE 'foo%';
  ```

- create trigram index (GIN index using operator class provided by `pg_trgm` module)

  - <https://www.postgresql.org/docs/9.6/static/pgtrgm.html>
  - (!) <http://blog.scoutapp.com/articles/2016/07/12/how-to-make-text-searches-in-postgresql-faster-with-trigram-similarity>

  all kinds of patterns (not only left-anchored ones) are supported by
  trigram indexes => it makes them good for general searches.

  > PostgreSQL splits a string into words and determines trigrams for each
  > word separately. It also normalizes the word by downcasing it, prefixing
  > two spaces and suffixing one. Non-alphanumeric characters are considered
  > to be word boundaries. Note that downcasing makes trigrams case-insensitive.
  >
  > We don't need any special changes to the query to use the trigram index.
  > Any query using LIKE will improve.

  ```sql
  CREATE EXTENSION pg_trgm;
  CREATE INDEX users_on_name_idx ON users USING GIN (name gin_trgm_ops);
  ```

- create combined multicolumn GIN index

  - <https://stackoverflow.com/a/29414489>
  - <https://stackoverflow.com/questions/40409997>
  - <https://www.postgresql.org/docs/current/static/btree-gin.html>

  this is useful when you want to search on some column using B-tree index
  and another column using trigram index (that is using `LIKE` operator).

  > for queries that test both a GIN-indexable column and a B-tree-indexable
  > column, it might be more efficient to create a multicolumn GIN index that
  > uses one of these operator classes (operator classes provided by btree_gin
  > module - my note) than to create two separate indexes that would have to be
  > combined via bitmap ANDing.


  ```sql
  CREATE EXTENSION pg_trgm;
  CREATE EXTENSION btree_gin;
  CREATE INDEX teams_on_name_idx ON teams USING GIN (name gin_trgm_ops, city);
  ```

  > if you install btree_gin, you can create a GIN index over “basic” data types
  > like integer, varchar or text.

  if B-tree-indexable column has another data type (say, boolean) you'll have
  to create 2 separate indexes:

  ```sql
  CREATE INDEX teams_on_name_idx ON teams USING GIN (name gin_trgm_ops);
  CREATE INDEX teams_on_is_public_idx ON teams (is_public);
  ```

## create/restore backup

### with data only

```sh
$ pg_dump -aOf dump.sql database
```

```sh
$ psql -U username -f ./dump.sql database
```

### using `backup` gem

```sh
$ RAILS_ENV=development bundle exec backup perform -t model_name -c ./config.rb
```

```sh
/ download backup archive
$ tar xvf model_name.tar
$ cd model_name/databases/
$ gunzip PostgreSQL.sql.gz
$ psql -U username -d database -f ./PostgreSQL.sql
```

## [how to] remove all versions of PostgreSQL on Ubuntu

<https://askubuntu.com/a/32735>:

```sh
$ sudo apt-get --purge remove postgresql postgresql-doc postgresql-common
```
