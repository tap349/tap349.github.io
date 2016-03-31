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

## bundler

- <http://bundler.io/man/bundle-exec.1.html>:

> bundle exec command executes the command, making all gems specified
> in the Gemfile(5) available to require in Ruby programs.

also bundler is used to manage application dependencies (gemsets) -
RVM provides functionality to do it as well but bundler is a better way.
rbenv doesn't manage gemsets in contrast with RVM.

there might be multiple versions of the same gem installed for particular
ruby version but project uses gem versions from bundle
(specified in _Gemfile.lock_).

when running commands (`rake`, etc.) from within project directory it's necessary
to use gem versions from bundle - this can be done in 2 ways:

- run commands with `bundle exec`
- use bundler binstubs

### bundle exec

using `bundle exec` is okay except that it's necessary to type it whenever
you run gem commands:

- alias common commands to be run with `bundle exec` in _.zshrc_:

  ```sh
  alias rails='bundle exec rails'
  alias rake='bundle exec rake'
  alias rspec='bundle exec rspec'
  ```

  also make sure these commands are run with `bundle exec` in other aliases
  (alias definitions are non-recursive - like `noremap` mappings in vim).

- run `rspec` with `bundle exec` in _Guardfile_:

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

to make rbenv aware of bundler binstubs use
[rbenv-binstubs](https://github.com/ianheggie/rbenv-binstubs) rbenv plugin.

```sh
$ rbenv which rspec
/Users/tap/.rbenv/versions/2.3.0/bin/rspec
```

## spring

<https://github.com/rails/spring>

when either using `bundle exec` or bundler binstubs
commands are NOT RUN via spring preloader!
(well, except for `bundle exec rails console`
but this is just a weird exception)

**NOTE**: don't use spring in production!

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

  **NOTE**: it's not possible to run other commands with spring.

  spring binstubs are run in context of bundle -
  no need to use `bundle exec` or bundler binstubs.
  don't use bundler and spring binstubs at the same time since they are
  both stored in _bin_ directory by default and might overwrite each other.

## okay, wtf to do with all this stuff about bundler and spring?

- generate spring binstubs for `rails`, `rake` and `rspec`
- use `rspec` spring binstub in _Guardfile_:

  ```ruby
  guard :rspec, cmd: 'bin/rspec -f doc', failed_mode: :none do
    ...
  end
  ```
- run `guard` as is - without any binstubs or `bundle exec`
- add these (or similar) aliases in _.zshrc_:

  ```sh
  alias rails='bin/rails'
  alias rake='bin/rake'
  alias rspec='bin/rspec'

  alias r='bin/rails'
  alias rc='bin/rails console'
  alias rd='bin/rails dbconsole'
  alias rs='bin/rails server'

  alias migrate='bin/rake db:migrate && RAILS_ENV=test bin/rake db:migrate'
  alias rollback='bin/rake db:rollback && RAILS_ENV=test bin/rake db:rollback'

  alias sidekiq='bundle exec sidekiq --config ./config/sidekiq.yml'
  ```

  **NOTE**: run sidekiq with `bundle exec` because there is no spring binstub for it.

