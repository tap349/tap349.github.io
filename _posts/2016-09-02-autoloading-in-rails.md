---
layout: post
title: autoloading in Rails
date: 2016-09-02 18:27:48 +0300
access: public
categories: [rails]
---

using non-standard directories in Rails project (_lib/_, _system/_, etc.).

<!-- more -->

## autoload_paths and eager_load_paths

- <http://urbanautomaton.com/blog/2013/08/27/rails-autoloading-hell>
- <http://blog.arkency.com/2014/11/dont-forget-about-eager-load-when-extending-autoload>

by default only subdirectories of _app/_ are added to `autoload_paths` and
`eager_load_paths` (the latter is used in environments with eager loading -
staging or production).

other directories (say, _lib/_) added manually to `autoload_paths` are not
added to `eager_load_paths` automatically - it's necessary to do it explicitly
in _config/application.rb_:

```ruby
config.autoload_paths += Dir["#{config.root}/lib/**/"]
config.eager_load_paths += Dir["#{config.root}/lib"]
```

the same in one go (note it's not necessary to add all nested directories):

```ruby
config.paths.add 'lib', eager_load: true
```

not adding them to `eager_load_paths` not only causes performance issues
(classes inside _lib/_ are lazy loaded in production) but also might be the
source of weird uninitialized constant errors.

to sum up **always** add non-standard directories to both `autoload_paths`
and `eager_load_paths` to avoid loading errors in production.

## using modules to namespace classes

2 ways to organize code in non-standard directories:

- always use fully qualified class names:

  ```ruby
  class Forms::Google::Adwords::Config < Reform::Form
    ...
  ```

  this allows to avoid loading problems but might be very inconvenient when
  class has a lot of modules and inherits from another class with the same
  amount of modules. also this way requires to use fully qualified names
  when referencing other classes from inside class definition.

- nest class name inside modules:

  ```ruby
  module Forms
    module Google
      module Adwords
        class Config < Reform::Form
          ...
  ```

  note if project has some class with `Config` module you must inline it
  (use it as a part of class name):

  ```ruby
  module Operations
    module Google
      module Adwords
        class Config::Create < Operations::Base
          ...
  ```

  since otherwise:

  ```ruby
  module Operations
    module Google
      module Adwords
        module Config
          class Create < Operations::Base
            ...
  ```

  Rails might complain that `Config` is not a class if it loads
  `Operations::Google::Adwords::Config::Create` first and knows that
  `Config` is a module but later you try to define it as a class.

  in some cases when top-level modules don't overlap you can type them
  side by side:

  ```ruby
  module Persistence::Repositories
    module Google
      module Adwords
        class Configs < Base[:google_adwords_configs]
          ...
  ```
