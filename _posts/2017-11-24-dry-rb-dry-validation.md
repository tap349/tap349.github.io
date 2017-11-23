---
layout: post
title: dry-rb - dry-validation (the missing guide)
date: 2017-11-24 00:17:19 +0300
access: public
comments: true
categories: [dry-rb]
---

<!-- more -->

* TOC
{:toc}
<hr>

dry-rb documentation is great but still the docs on dry-validation
has always made me crazy since I couldn't discern a pattern in how
its DSL is organized and when to use of these variants:

`type?(Integer)` vs. `type?: Integer` vs. `:int?` vs. `{ int? }` vs. anything else?

still not confused? we can use `int?` predicate inside `filled` macro
which is a predicate itself (`filled?`) like this: `filled(:int?)`.

authors of dry-rb claim their libraries to be dry and simple but all this
DSL madness looks like black magic much more than any Rails gem does.

frankly speaking, these patterns are not that hard to recognize but still
I wish they were stated in documentation more clearly.

## `value` vs. block

a predicate can be specified as (both ways are equivalent):

- an argument (atom or hash) of `value` method

  ```ruby
  required(:foo).value(:int?)
  ```

- a method call inside block passed to `required` method

  ```ruby
  required(:foo) { int? }
  ```

## predicates with argument

predicates might have arity of 0 (`filled?`) or 1 (`gt?`).

this is how argument is passed to a unary predicate:

```ruby
# when using `value` method
required(:foo).value(type?: Integer)
# when using block
required(:foo) { type?(Integer) }
```

NOTE: when using `value` method, you pass hash for unary predicate
      instead of atom for predicate of zero arity.

## `int?` vs. `type?(Integer)`

<http://dry-rb.org/gems/dry-validation/basics/built-in-predicates/>:

> `int?` is a shorthand for `type?(Integer)`

NOTE: predicates have different arity.

## multiple predicates

when custom predicate logic is required, using blocks is the only option
but if it's necessary just to AND multiple predicates together (the most
common case I guess) this can be done using `value` method as well:

```ruby
# when using `value` method
required(:foo).value(:str?, min_size?: 3)
# when using block
required(:foo) { str? & min_size?(3) }
```

## custom type

1. <https://github.com/dry-rb/dry-validation/issues/161#issuecomment-232333065>
2. <https://gist.github.com/AMHOL/0671986632fe734189c4c73e2a665f8b>

it's possible to register custom class (say, AR model) as a dry type
(in the same dry container where built-in types are registered) so that
it can be used in validation rules afterwards:

```ruby
schema = Dry::Validation.Schema do
  Dry::Types.register_class(User)
  required(:user).filled(type?: Types::User)
end
```

or just:

```ruby
schema = Dry::Validation.Schema do
  required(:user).filled(type?: User)
end
```

## macros

1. <http://dry-rb.org/gems/dry-validation/basics/macros/>

macros don't share a common pattern - just memorize how they are expanded
(e.g. that `filled(:int?)` == `{ filled? & int? }`).
