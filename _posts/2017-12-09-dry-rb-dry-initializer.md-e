---
layout: post
title: dry-rb - dry-initializer
date: 2017-12-09 04:02:03 +0300
access: public
comments: true
categories: [dry-rb]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

optional attributes and default values
--------------------------------------

1. <http://dry-rb.org/gems/dry-initializer/optionals-and-defaults>

- optional attribute

  ```ruby
  # option can be not specified at all
  option :foo, Types::Strict::String, optional: true
  ```

- default value

  when using `default` key, `optional: true` is implied:

  ```ruby
  # equivalent to `Types::Strict::String, optional: true`
  option :foo, Types::Strict::String, default: proc { nil }
  option :foo, Types::Strict::String, default: proc { 'bar' }
  ```

  NOTE: the last example is not equivalent to:

  ```ruby
  option :foo, Types::Strict::String.default('bar'), optional: true
  ```

  such usage is incorrect - when option is not specified, it will be `nil`.
