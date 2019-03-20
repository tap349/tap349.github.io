---
layout: post
title: Debian - Commands
date: 2017-09-10 17:49:12 +0300
access: public
comments: true
categories: [debian, ubuntu]
---

of course these commands should work on Ubuntu as well.

<!-- more -->

* TOC
{:toc}
<hr>

show/list
---------

#### show package info

```sh
$ apt-cache show <packagename>
```

#### list all files of specified package

```sh
$ sudo dpkg-query -L <packagename>
```

#### list package versions

1. <https://www.cyberciti.biz/faq/debian-ubuntu-linux-apt-get-aptitude-show-package-version-command/>

```sh
$ apt-cache policy elixir
elixir:
  Installed: 1.5.1-1
  Candidate: 1.5.1-1
  Version table:
  *** 1.5.1-1 500
        500 http://packages.erlang-solutions.com/ubuntu xenial/contrib amd64 Packages
        100 /var/lib/dpkg/status
      1.5.1-1 500
        500 http://packages.erlang-solutions.com/ubuntu xenial/contrib i386 Packages
      1.5.0-1 500
        500 http://packages.erlang-solutions.com/ubuntu xenial/contrib amd64 Packages
      1.5.0-1 500
        500 http://packages.erlang-solutions.com/ubuntu xenial/contrib i386 Packages
```

find
----

#### find any file

```sh
$ sudo updatedb
$ sudo locate <filename>
```

#### find package by name

```sh
$ apt-cache search <packagename>
```

#### find package containing specified file

```sh
$ sudo dpkg-query -S <filename>
```

update/upgrade
--------------

#### update (synchronize package index files)

```sh
$ sudo apt update
```

#### upgrade (upgrade all packages)

```sh
$ sudo apt upgrade
```

remove
------

#### remove package (only binaries)

```sh
$ sudo apt remove <packagename>
```

#### remove specific version of package

```sh
$ sudo apt remove <packagename>=<version>
```

it's possible to use glob pattern instead of exact version (say, if you don't
know exact version currently installed):

```sh
$ sudo apt remove postgresql=10*
```

#### remove everything regarding package but without dependencies

```sh
$ sudo apt [purge|remove --purge] <packagename>
```

#### remove everything regarding package with dependencies

```sh
$ sudo aptitude [remove|purge] <packagename>
```

#### remove all orphaned packages

```sh
$ sudo apt autoremove
```
