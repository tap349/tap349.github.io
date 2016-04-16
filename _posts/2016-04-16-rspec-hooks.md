---
layout: post
title: rspec before hooks
date: 2016-04-16 15:49:59 +0300
access: public
categories: [rspec]
---

<http://rspec.info/blog/2014/05/notable-changes-in-rspec-3/>

RSpec before hooks (usually configured in _spec/rails_helper.rb_):

- `:each` (aliased to `:example` in RSpec 3):

  runs before each example (before each `it` test).

- `:all` (aliased to `:context` in RSpec 3):

  runs before the 1st example of each top-level example group
  (before each top-level `describe` in spec files).

- `:suite`:

  runs once after all spec files have been loaded, before the 1st spec runs
  (once for each rspec run - regardless of number of spec files in current run).

