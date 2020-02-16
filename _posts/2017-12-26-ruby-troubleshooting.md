---
layout: post
title: Ruby - Troubleshooting
date: 2017-12-26 19:23:05 +0300
access: public
comments: true
categories: [ruby]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

the error occurred after installing a new Ruby version (2.5.0):

```
$ bundle
...
An error occurred while installing json (1.8.3), and Bundler cannot continue.
Make sure that `gem install json -v '1.8.3'` succeeds before bundling.
```

**solution**

try to update offending gem:

```sh
$ bundle update json
```

## uninitialized constant Gem::BundlerVersionFinder (NameError)

```
$ berks
...
/Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems/dependency.rb:283:in `matching_specs': uninitialized constant Gem::BundlerVersionFinder (NameError)
```

**solution**

error was gone after upgrading Ruby version from 2.5.0 to 2.5.1:

```
$ echo 2.5.1 > .ruby-version
```

if this doesn't help, try to clean up old versions of installed gems (this will
remove currently installed versions as well):

```sh
$ gem cleanup
$ bundle
```

`gem cleanup` removes bundled gems only (gems specified in _Gemfile_ of current
project) though it might affect other projects if they are using the same gems
(run `bundle` for those projects as well in that case).

**_UPDATE_**

it has turned out the problem was caused by specific version of either
`berkshelf` or `chef` or both.

old versions (error is present):

```ruby
gem 'berkshelf', '6.3.1'
gem 'chef', '13.5.3'
```

old versions (error is gone):

```ruby
gem 'berkshelf', '7.0.2'
gem 'chef', '14.1.1'
```

## can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)

```
$ bundle
Traceback (most recent call last):
	2: from /Users/tap/.rbenv/versions/2.5.0/bin/bundle:23:in `<main>'
	1: from /Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
/Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
```

**solution**

1. <https://stackoverflow.com/a/54038218/3632318>

find current Bundler version:

```
$ gem list --local bundler

*** LOCAL GEMS ***

bundler (2.0.2)
```

find Bundler version in _Gemfile.lock_:

```
# Gemfile.lock

BUNDLED WITH
  1.16.1
```

now there are 2 possible solutions:

### sync Gemfile.lock to Bundler version

```diff
  # Gemfile.lock

  BUNDLED WITH
-   1.16.1
+   2.0.1
```

### sync Bundler version to Gemfile.lock

1. <https://bundler.io/blog/2019/01/04/an-update-on-the-bundler-2-release.html>

```sh
$ gem install bundler -v 1.16.1
```

in many cases this solution is preferable since it's easier and causes less
problems with CI and deployment.

## [__NSCFConstantString initialize] may have been in progress in another thread when fork() was called

```
$ rails console
[1] pry(main)> Profile.count
objc[29838]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called.
objc[29838]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead. Set a breakpoint on objc_initializeAfterForkError to debug.
```

**solution**

1. <https://github.com/puma/puma/issues/2084>

error has something to do with the latest version of `psql` (12.1) that must be
used by `pg` gem (`psql` is provided by `libpq` package in my case).

- edit `libpq` formula to install old version (11.7)

  ```sh
  $ brew edit libpq
  ```

  ```diff
    # /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/libpq.rb

    class Libpq < Formula
      desc "Postgres C API library"
      homepage "https://www.postgresql.org/docs/12/libpq.html"
  -   url "https://ftp.postgresql.org/pub/source/v12.1/postgresql-12.1.tar.bz2"
  -   sha256 "a09bf3abbaf6763980d0f8acbb943b7629a8b20073de18d867aecdb7988483ed"
  +   url "https://ftp.postgresql.org/pub/source/v11.7/postgresql-11.7.tar.bz2"
  +   sha256 "324ae93a8846fbb6a25d562d271bc441ffa8794654c5b2839384834de220a313"
      revision 1
  ```

- reinstall `libpq` package from source

  ```sh
  $ brew reinstall libpq --build-from-source
  $ psql --version
  psql (PostgreSQL) 11.7
  ```

  NOTE: maybe it's not necessary to build from source.

- reinstall `pg` gem to build native extensions against old version of `libpq`

  ```diff
    # Gemfile

  - gem 'pg'
  + #gem 'pg'
  ```

  ```sh
  $ bundle
  ```

  ```diff
    # Gemfile

  - #gem 'pg'
  + gem 'pg'
  ```

  ```sh
  $ bundle
  ```

- rollback changes in `libpq` formula

  ```sh
  $ cd /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core
  $ git checkout Formula/libpq.rb
  ```

- reinstall `libpq` package

  ```sh
  $ brew reinstall libpq
  $ psql --version
  psql (PostgreSQL) 12.1
  ```
