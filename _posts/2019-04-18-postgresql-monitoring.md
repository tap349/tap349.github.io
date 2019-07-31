---
layout: post
title: PostgreSQL - Monitoring
date: 2019-04-18 11:53:30 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## tools

- `psql`
- pgAdmin

  ```sh
  $ brew cask install pgadmin4
  ```

- `pg_top`

  > man pg_top
  >
  > display and update information about the top cpu PostgreSQL processes

  ```sh
  $ brew install pg_top
  $ pg_top -h localhost -p 5434 -U postgres -d alice_dev -W
  / enter postgres password
  ```

## open sessions (connections)

1. <https://serverfault.com/questions/128284>

- `psql`

  ```sql
  SELECT * FROM PG_STAT_ACTIVITY WHERE DATNAME = 'MY_DATABASE';
  ```

  the number of connections currently used:

  ```sql
  SELECT COUNT(*) FROM PG_STAT_ACTIVITY WHERE DATNAME = 'MY_DATABASE';
  ```

- pgAdmin

  | pgAdmin                                                                     |
  | --------------------------------------------------------------------------- |
  | `Servers` (browser) → `<MY_SERVER>` → `Databases` → `<MY_DATABASE>`         |
  | `Dashboard` (tab in main window) → `Server activity` (section) → `Sessions` |
