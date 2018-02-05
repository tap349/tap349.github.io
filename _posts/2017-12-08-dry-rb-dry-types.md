---
layout: post
title: dry-rb - dry-types
date: 2017-12-08 22:51:34 +0300
access: public
comments: true
categories: [dry-rb]
---

<!-- more -->

* TOC
{:toc}
<hr>

custom types
------------

[old way](https://gist.github.com/AMHOL/0671986632fe734189c4c73e2a665f8b):

```ruby
Dry::Types.register_class(User)
param :user, Types::User
```

[new way](http://dry-rb.org/gems/dry-types/custom-types/):

```ruby
param :user, Types.Instance(User)
```

optional and default values
---------------------------

1. <http://dry-rb.org/gems/dry-types/optional-values/>
2. <http://dry-rb.org/gems/dry-types/default-values/>

- optional value

  use optional type (sum type under the hood):

  ```ruby
  # value can be nil
  Types::Strict::String.optional
  # which is syntactic sugar for
  Types::Strict::Nil | Types::Strict::String
  ```

  maybe type is kind of advanced version of optional type
  (`Maybe` types wrap passed values in `Maybe` monad).

- default value

  ```ruby
  # value is 'bar' when input is nil
  # (final value can't be nil => `default(nil)` is not allowed)
  Types::Strict::String.default('bar')
  ```
