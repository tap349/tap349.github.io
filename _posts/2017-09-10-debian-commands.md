---
layout: post
title: Debian - Commands
date: 2017-09-10 17:49:12 +0300
access: public
categories: [debian, ubuntu]
---

of course these commands should work on Ubuntu as well.

<!-- more -->

* TOC
{:toc}
<hr>

- locate any file:

  ```sh
  $ sudo updatedb
  $ sudo locate <filename>
  ```

- list all files of specified package

  ```sh
  $ sudo dpkg-query -L <packagename>
  ```

- search for packages containing specified file

  ```sh
  $ sudo dpkg-query -S <filename>
  ```

- remove package

  remove only binaries:

  ```sh
  $ sudo apt remove <packagename>
  ```

  remove everything regarding package but without dependencies:

  ```sh
  $ sudo apt [purge|remove --purge] <packagename>
  ```

  remove all orphaned packages:

  ```sh
  $ sudo apt autoremove
  ```

  remove everything regarding package with dependencies:

  ```sh
  $ sudo aptitude [remove|purge] <packagename>
  ```

- list package versions

  <https://www.cyberciti.biz/faq/debian-ubuntu-linux-apt-get-aptitude-show-package-version-command/>:

  ```sh
  $ sudo apt-cache policy elixir
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
