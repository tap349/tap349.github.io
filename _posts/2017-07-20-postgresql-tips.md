---
layout: post
title: PostgreSQL - Tips
date: 2017-07-20 16:09:32 +0300
access: public
comments: true
categories: [postgresql, rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

psql tips
---------

1. <https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS>

### login

```sh
$ psql -h localhost -U sith_prod sith_prod
/ enter password for database user sith_prod
```

as `postgres` user:

```sh
$ sudo -u postgres psql -U postgres sith_prod
```

### users

- list roles (= users) and their attributes

  ```
  \u
  ```

### databases

- connect to database

  ```
  \c sith_dev
  ```

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
  $ psql -d sith_prod
  > CREATE EXTENSION pg_trgm;
  ```

query optimization
------------------

1. <https://robots.thoughtbot.com/why-postgres-wont-always-use-an-index>

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

1. <https://stackoverflow.com/questions/1566717>

- create B-tree index

  only left-anchored patterns (no leading wildcard) are supported by
  B-tree indexes => they are good for auto-completion.

  ```sql
  SELECT * FROM tbl WHERE col LIKE 'foo%';
  ```

- create trigram index (GIN index using operator class provided by `pg_trgm` module)

  1. <https://www.postgresql.org/docs/9.6/static/pgtrgm.html>
  2. (!) <http://blog.scoutapp.com/articles/2016/07/12/how-to-make-text-searches-in-postgresql-faster-with-trigram-similarity>

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

  1. <https://stackoverflow.com/a/29414489>
  2. <https://stackoverflow.com/questions/40409997>
  3. <https://www.postgresql.org/docs/current/static/btree-gin.html>

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

backup
------

### using `pg_dump`

create backup with data only:

```sh
(remote)$ mkdir ~/tmp && cd ~/tmp
(remote)$ pg_dump -h localhost -U sith_prod -aOf dump.sql sith_prod
(remote)$ tar cvzf dump.sql.tar.gz dump.sql
```

create backup with structure and data:

```sh
(remote)$ mkdir ~/tmp && cd ~/tmp
(remote)$ pg_dump sith_prod > dump.sql
(remote)$ tar cvzf dump.sql.tar.gz dump.sql
```

NOTE: use `postgres` user when creating backup on macOS:

```sh
(local)$ pg_dump -h localhost -U postgres -aOf dump.sql sith_dev
```

restore:

```sh
(local)$ cd ~/tmp
(local)$ scp ssh_host:~/tmp/dump.sql.tar.gz dump.sql.tar.gz
(local)$ tar xvzf dump.sql.tar.gz
(local)$ psql -U sith_prod -f dump.sql sith_prod
```

this dump doesn't create tables so make sure schema is the same -
if it's not the case it might be necessary to reset database with
required schema. say, in Rails project:

```sh
$ git checkout master
$ rails db:reset
```

### [Rails] using `backup` gem (structure + data)

create:

```sh
$ RAILS_ENV=production bundle exec backup perform -t model_name -c ./config.rb
```

restore:

```sh
/ download backup archive
$ tar xvf model_name.tar
$ cd model_name/databases/
$ gunzip PostgreSQL.sql.gz
$ psql -U username -f ./PostgreSQL.sql database
$ RAILS_ENV=test rails db:structure:load
```

say, to restore our database running in Docker:

```sh
$ psql -h localhost -p 5433 -U postgres -f ./PostgreSQL.sql sith_dev
```

(how to) remove all versions of PostgreSQL on Ubuntu
----------------------------------------------------

<https://askubuntu.com/a/32735>:

```sh
$ sudo apt-get --purge remove postgresql postgresql-doc postgresql-common
```

(how to) move local database into Docker container
--------------------------------------------------

1. <https://github.com/wsargent/docker-cheat-sheet>

### add `db` service

_docker-compose.yml_:

```diff
  services:
+   db:
+     image: postgres:9.4
+     environment:
+       POSTGRES_PASSWORD: sith_dev
+     ports:
+       - 5433:5432
```

### run `db` service

```sh
$ docker-compose up db
```

### configure application to use databases in Docker container

_config/database.yml_:

```diff
  default: &default
-   port: 5432
+   port: 5433

  development:
-   username: sith_dev
+   username: postgres

  test:
-   username: sith_test
+   username: postgres
```

`postgres` is a default superuser in PostgreSQL image - it can be changed
by setting `POSTGRES_USER` environment variable in _docker-compose.yml_.

### create new databases in Docker container

```sh
$ rails db:create
Created database 'sith_dev'
Created database 'sith_test'
```

### import local database into Docker container

- using `pg_dump` (structure + data)

  ```sh
  $ DB_CONTAINER_ID=`docker-compose ps -q db`
  $ DB_USER=postgres
  $ DB_NAME=sith_dev
  $ pg_dump -h localhost "${DB_NAME}" -p 5432 | \
      docker exec -i "${DB_CONTAINER_ID}" \
      psql -U "${DB_USER}" -d "${DB_NAME}" -v ON_ERROR_STOP=1
  / or
  $ pg_dump -h localhost "${DB_NAME}" -p 5432 | \
      docker-compose exec -T db \
      psql -U "${DB_USER}" -d "${DB_NAME}" -v ON_ERROR_STOP=1
  ```

  load test database:

  ```sh
  $ RAILS_ENV=test rails db:structure:load
  ```

- using _db/structure.sql_ (structure only)

  command `rails db:structure:load` doesn't work (some error) when database is
  inside Docker container but still it's possible to load _db/structure.sql_:

  ```sh
  $ DB_CONTAINER_ID=`docker-compose ps -q db`
  $ DB_USER=postgres
  $ DB_NAME=sith_dev
  $ cat db/structure.sql | docker exec -i "${DB_CONTAINER_ID}" \
      psql -U "${DB_USER}" -d "${DB_NAME}" -v ON_ERROR_STOP=1
  / or
  $ cat db/structure.sql | docker-compose exec -T db \
      psql -U "${DB_USER}" -d "${DB_NAME}" -v ON_ERROR_STOP=1
  ```

  in case there are errors executing previous command:

  - remove `-v ON_ERROR_STOP=1` psql option to skip errors

    ```sh
    $ cat db/structure.sql | docker exec -i "${DOCKER_DB_NAME}" \
        psql -U "${DB_USER}" -d "${DB_NAME}"
    / or
    $ cat db/structure.sql | docker-compose exec -T db \
        psql -U "${DB_USER}" -d "${DB_NAME}"
    ```

  - drop and create test database if skipping errors doesn't help

    ```sh
    $ RAILS_ENV=test rails db:drop
    $ RAILS_ENV=test rails db:create
    / run command to load structure.sql
    ```

### run psql

list Docker containers:

```sh
$ docker-compose ps
          Name                        Command               State                           Ports
--------------------------------------------------------------------------------------------------------------------------
sith_db_1                  docker-entrypoint.sh postgres    Up      0.0.0.0:5433->5432/tcp
```

run psql in a running Docker container:

```sh
$ docker exec -it sith_db_1 psql -U 'postgres' -d sith_dev
```

(how to) get timezone offset
----------------------------

```sql
SELECT * FROM pg_timezone_names WHERE name='Europe/Moscow';
```
