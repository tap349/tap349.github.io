---
layout: post
title: bundler and spring
date: 2016-03-31 23:17:24 +0300
access: public
categories: [bundler, spring, rbenv]
---

some background information about bundler and spring as well as some
intricacies of using `bundle exec`, bundler and spring binstubs.

<!-- more -->

* TOC
{:toc}

<hr>

## bundler

<http://bundler.io/man/bundle-exec.1.html>:

> bundle exec command executes the command, making all gems specified
> in the Gemfile(5) available to require in Ruby programs.

bundler setup for Rails application takes place in _config/boot.rb_:

```ruby
ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../../Gemfile', __FILE__)
require 'bundler/setup' # Set up gems listed in the Gemfile.
```

also bundler is used to manage application dependencies (gemsets) -
RVM provides functionality to do it as well but bundler is a better way.
rbenv doesn't manage gemsets at all in contrast to RVM.

there might be multiple versions of the same gem installed for particular ruby
version but project uses gem versions from bundle (specified in _Gemfile.lock_).

when running commands (`rake`, etc.) from within project directory
it's necessary to use bundle gem versions - this can be done in 2 ways:

- run commands with `bundle exec`
- use bundler binstubs

### bundle exec

using `bundle exec` is okay except that it's necessary to type it whenever
you run gem commands:

- alias common commands to be run with `bundle exec`

  _.zshrc_:

  ```sh
  alias rails='bundle exec rails'
  alias rake='bundle exec rake'
  alias rspec='bundle exec rspec'

  # alias definitions are recursive:
  # other aliases will expand `rspec` to `bundle exec rspec`
  # (previously I thought that they are not)

  alias cap='bundle exec cap'
  alias guard='bundle exec guard'
  alias sidekiq='bundle exec sidekiq --config ./config/sidekiq.yml'
  ```

- run `rspec` with `bundle exec` in _Guardfile_

  _Guardfile_:

  ```ruby
  guard :rspec, cmd: 'bundle exec rspec -f doc', failed_mode: :none do
    ...
  end
  ```

### bundler binstubs

alternatively generate bundler binstubs:

```sh
$ bundle install --binstubs
```

to make rbenv is aware of bundler binstubs use
[rbenv-binstubs](https://github.com/ianheggie/rbenv-binstubs) rbenv plugin.

```sh
$ rbenv which rspec
/Users/tap/.rbenv/versions/2.3.0/bin/rspec
```

## spring

<https://github.com/rails/spring>

when using `bundle exec` or bundler binstubs commands are not run with spring!
(well, except for `bundle exec rails console` but this is just a weird exception)

NOTE: don't use spring in production!

to use spring preloader:

- run commands with `bundle exec spring` or just `spring`

  in previous releases of spring there was a warning about
  significant performance penalty when running spring with bundler
  (`bundle exec spring`) - probably this is not the case now.

- use spring binstubs

  **RECOMMENDED WAY**

  command `spring binstub --all` generates spring binstubs for 3 commands:

  - `rails`
  - `rake`
  - `rspec`

  also it generates _bin/spring_ which is loaded in each of these binstubs.
  in fact `bin/rails` is equivalent to `bin/spring rails`.

  NOTE: it's not possible to run other commands with spring.

  spring binstubs are run in context of a bundle -
  there is no need to use `bundle exec` or bundler binstubs.

  don't use bundler and spring binstubs at the same time since they are
  both stored in _bin/_ directory by default and might overwrite each other.

## okay, wtf to do with all this stuff about bundler and spring?

- generate spring binstubs for `rails`, `rake` and `rspec`
- use `rspec` spring binstub in _Guardfile_

  _Guardfile_:

  ```ruby
  guard :rspec, cmd: 'bin/rspec -f doc', failed_mode: :none do
    ...
  end
  ```

- add these (or similar) aliases in _.zshrc_:

  ```sh
  alias rails='bin/rails'
  alias rake='bin/rake'
  alias rspec='bin/rspec'

  # other aliases with rails, rake and rspec inside
  # must use their spring binstub counterparts as well -
  # alias definitions are recursive

  alias cap='bundle exec cap'
  alias guard='bundle exec guard'
  alias sidekiq='bundle exec sidekiq --config ./config/sidekiq.yml'
  ```

  sidekiq and Guard are run with `bundle exec` because there are no
  spring binstubs for them (and it doesn't make sense to make one
  for Guard since it's using `rspec` spring binstub inside) - still
  we need to run their bundle versions.
