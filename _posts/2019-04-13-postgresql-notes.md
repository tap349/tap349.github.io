---
layout: post
title: PostgreSQL - Notes
date: 2019-04-13 10:35:22 +0300
access: public
comments: true
categories: [postgresql]
---

<!-- more -->

* TOC
{:toc}
<hr>

session vs. connection
----------------------

> <https://stackoverflow.com/a/46323144/3632318>
>
> a session is "synonymous with a TCP connection"

custom configuration
--------------------

add custom configuration in _postgresql.conf_ and _pg_hba.conf_ files:

> <https://www.endpoint.com/blog/2010/09/27/postgres-configuration-best-practices>
>
> Put all important variables at the bottom, sans comments, one per line

> <https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server#Important_notes_about_configuration_files>
>
> If the same setting is listed multiple times, the last one wins.

or else it's possible to include custom configuration from a separate file but
this works for _postgresql.conf_ only:

> <https://www.postgresql.org/docs/current/config-setting.html>
>
> PostgreSQL provides several features for breaking down complex postgresql.conf
> files into sub-files.
>
> postgresql.conf file can contain `include` directives, which specify another
> file to read and process as if it were inserted into the configuration file at
> this point.
