---
layout: post
title: PostgreSQL - Tips
date: 2017-07-20 16:09:32 +0300
access: public
comments: true
categories: [postgresql, rails]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

psql
----

1. <https://www.postgresql.org/docs/current/static/app-psql.html#APP-PSQL-META-COMMANDS>

### login

```sh
$ psql -h localhost -U USERNAME DATABASE
/ enter password for database user USERNAME
```

as `postgres` user:

```sh
$ sudo -u postgres psql -U postgres DATABASE
```

### users

> <http://www.postgresqltutorial.com/postgresql-roles>
>
> A role can be a user or a group, depending on how you setup the role.
> A role that has login right is called user.
> A role may be a member of other roles, which are known as groups.

- list users (roles) and their attributes

  ```
  \du
  ```

- create user

  ```sql
  CREATE ROLE reporter WITH LOGIN PASSWORD 'password';
  ```

  > <https://www.postgresql.org/docs/current/sql-createuser.html>
  >
  > CREATE USER is now an alias for CREATE ROLE. The only difference is that
  > when the command is spelled CREATE USER, LOGIN is assumed by default,
  > whereas NOLOGIN is assumed when the command is spelled CREATE ROLE.

- drop user

  first remove objects owned by user (say, granted privileges):

  ```sql
  DROP OWNED BY reporter RESTRICT;
  ```

  then drop user himself:

  ```sql
  DROP USER reporter;
  ```

  > <https://www.postgresql.org/docs/current/sql-dropuser.html>
  >
  > DROP USER is simply an alternate spelling of DROP ROLE.

- grant privileges to user

  1. <https://stackoverflow.com/questions/760210>

  grant all privileges on all tables:

  ```sql
  GRANT ALL ON ALL TABLES IN SCHEMA PUBLIC TO reporter;
  ```

  grant `SELECT` privilege (read-only access) on all tables:

  ```sql
  GRANT SELECT ON ALL TABLES IN SCHEMA PUBLIC TO reporter;
  ```

- revoke privileges from user

  revoke all privileges on all tables:

  ```sql
  REVOKE ALL ON ALL TABLES IN SCHEMA PUBLIC TO reporter;
  ```

  revoke `SELECT` privilege (read-only access) on all tables:

  ```sql
  REVOKE SELECT ON ALL TABLES IN SCHEMA PUBLIC TO reporter;
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

- rename database

  ```sql
  ALTER DATABASE old_name RENAME TO new_name;
  ```

  it's also necessary to alter the owner of all non-system tables:

  ```sql
  ALTER TABLE old_name OWNER TO new_name;
  ```

### tables

- list all tables

  ```
  \d
  ```

  this commands also lists sequences. to list tables only:

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

  ```sql
  CREATE EXTENSION pg_trgm;
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

  it's not necessary to enable extension in migration itself (by adding
  `enable_extension :pg_trgm` statement) if it has already been created
  manually in `psql` - `enable_extension "btree_gin"` line will be added
  to _schema.rb_ after running migration anyway.

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

#### dump

there are 2 ways to create a dump file (say, _dump.sql_):

- send output to the dump file with `-f` option

  ```sh
  $ pg_dump -f dump.sql DATABASE
  ```

- redirect stdout to the dump file with `>` operator

  ```sh
  $ pg_dump DATABASE > dump.sql
  ```

```sh
$ pg_dump -h localhost -U USERNAME -p PORT -f dump.sql DATABASE
$ tar cvzf dump.sql.tar.gz dump.sql
```

NOTE: use `postgres` user on macOS.

use `-aO` options to dump the data only:

```sh
$ pg_dump -h localhost -U USERNAME -p PORT -aOf dump.sql DATABASE
$ tar cvzf dump.sql.tar.gz dump.sql
```

#### restore

```sh
$ tar xvzf dump.sql.tar.gz
$ psql -h localhost -U USERNAME -p PORT -f dump.sql DATABASE
```

it might be necessary to download the dump file from remote host first:

```sh
$ scp SSH_HOST:~/tmp/dump.sql.tar.gz dump.sql.tar.gz
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
$ psql -U USERNAME -f ./PostgreSQL.sql DATABASE
$ RAILS_ENV=test rails db:structure:load
```

say, to restore our database running in Docker:

```sh
$ psql -h localhost -p 5434 -U postgres -f ./PostgreSQL.sql sith_dev
```

(how to) remove all versions of PostgreSQL on Ubuntu
----------------------------------------------------

1. <https://askubuntu.com/a/32735>

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
+     image: postgres:11.1
+     environment:
+       POSTGRES_USER: postgres
+       POSTGRES_PASSWORD: postgres
+     ports:
+       - 5434:5432
```

### create databases in Docker container

Rails:

```sh
$ rails db:create
$ RAILS_ENV=test rails db:create
```

Phoenix:

```sh
$ mix ecto.create
$ MIX_ENV=test mix ecto.create
```

### import local database into Docker container

```sh
$ DB_USER=postgres
$ DB_NAME=reika_dev
$ DOCKER_PSQL_CMD='docker-compose exec -T db psql -U "${DB_USER}" -d "${DB_NAME}" -v ON_ERROR_STOP=1'
```

NOTE: remove `-v ON_ERROR_STOP=1` to skip errors (say, you'll get errors if
structure has been already loaded).

- using `pg_dump` (structure and data)

  ```sh
  $ pg_dump -h localhost "${DB_NAME}" -p 5432 | eval "${DOCKER_PSQL_CMD}"
  ```

- using structure file (structure only)

  Rails:

  ```sh
  $ cat db/structure.sql | eval "${DOCKER_PSQL_CMD}"
  ```

  Phoenix (dump structure when local database is used):

  ```sh
  $ cat priv/repo/structure.sql | eval "${DOCKER_PSQL_CMD}"
  ```

- load structure for test database

  Rails:

  ```sh
  $ RAILS_ENV=test rails db:structure:load
  ```

  Phoenix:

  ```sh
  $ mix ecto.dump
  $ MIX_ENV=test mix ecto.load
  ```

### run psql

```sh
$ psql -h localhost -p 5434 -U postgres -d reika_dev
```

(how to) get timezone offset
----------------------------

```sql
SELECT * FROM pg_timezone_names WHERE name='Europe/Moscow';
```

(how to) drop database
----------------------

the point is that you can't connect to the database you're going to drop -
you'll get the error then:

```
ERROR:  cannot drop the currently open database
```

connect to `postgres` database as superuser instead:

```
$ psql -U devops -d postgres -h localhost
postgres=# DROP DATABASE IF EXISTS eva_prod;
DROP DATABASE
```

## (how to) remove PostgreSQL completely

```sh
$ sudo apt autoremove --purge postgresql-12
$ sudo rm -rf /etc/postgresql
$ sudo rm -rf /run/postgresql
$ sudo rm -rf /var/lib/postgresql
$ sudo apt install postgresql-12
```
