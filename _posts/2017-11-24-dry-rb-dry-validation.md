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
have always made me crazy since I couldn't discern a pattern in how
its DSL is organized and when to use each of these variants:

`type?(Integer)` vs. `type?: Integer` vs. `:int?` vs. `{ int? }` vs. anything else?

still not confused? we can use `int?` predicate inside `filled` macro
which is a predicate itself (`filled?`) like this: `filled(:int?)`.

authors of dry-rb claim their libraries to be dry and simple but all this
DSL madness looks like black magic much more than any Rails gem does.

frankly speaking, these principles of organization are not that hard to
recognize but still I wish they were stated in documentation more clearly.

macro and block syntax
----------------------

usually some hash is validated: hash key is passed to `required`
method while predicates to validate hash value are specified in
one of 2 equivalent ways:

- by their name (atom and hash)

  predicate names are passed to some macro - `value` method
  looks like a macro as well (but most generic one):

  ```ruby
  required(:foo).value(:int?)
  ```

  this way will be referred to as `macro syntax`.

- by calling them as methods

  predicate methods are called inside block passed to
  `required` method (at least they look like method calls):

  ```ruby
  required(:foo) { int? }
  ```

  this way will be referred to as `block syntax`.

macros
------

1. <http://dry-rb.org/gems/dry-validation/basics/macros>

macros don't share a common pattern - just memorize how they are expanded:
(e.g. `filled(:int?)` => `{ filled? & int? }`).

I've mentioned `value` method that looks like a macro - it just applies all
passed predicates (joined by conjunction) without introducing any additional
logic (e.g. `value(:int?, min_size?: 3) => { int? & min_size?(3)}`).

it's possible to pass any Dry type into macro:

```ruby
required(:foo).filled(Types::Strict::String.enum('bar', 'baz'))
```

predicates with argument
------------------------

predicates might have arity of 0 (`filled?`) or 1 (`gt?`).

this is how argument is passed to a unary predicate:

```ruby
# macro syntax
required(:foo).value(type?: Integer)
# block syntax
required(:foo) { type?(Integer) }
```

NOTE: when using macro syntax, unary predicate is specified as
      a hash (predicate of zero arity is specified as an atom).

`int?` vs. `type?(Integer)`
---------------------------

> <http://dry-rb.org/gems/dry-validation/basics/built-in-predicates>
>
> `int?` is a shorthand for `type?(Integer)`

NOTE: the predicates have different arity.

multiple predicates
-------------------

when custom predicate logic is required, using block syntax is the only
option but if it's necessary just to AND multiple predicates together
(the most common case I guess) this can be done using macro syntax too:

```ruby
# macro syntax
required(:foo).value(:str?, min_size?: 3)
# block syntax
required(:foo) { str? & min_size?(3) }
```

custom type
-----------

1. <https://github.com/dry-rb/dry-validation/issues/161#issuecomment-232333065>
2. <https://gist.github.com/AMHOL/0671986632fe734189c4c73e2a665f8b>

it's possible to register custom class (say, AR model) as a dry type
(in the same dry container where built-in types are registered) so that
it can be used in validation rules:

```ruby
schema = Dry::Validation.Schema do
  Dry::Types.register_class(User)
  required(:user).filled(type?: Types::User)
end
```

or just (this also works!):

```ruby
schema = Dry::Validation.Schema do
  required(:user).filled(type?: User)
end
```

nested arrays
-------------

1. <http://dry-rb.org/gems/dry-validation/nested-data>
2. <https://github.com/dry-rb/dry-validation/issues/175>

```ruby
# array cannot be nil but can be empty
required(:ids).each(:int?)

# array cannot be nil or empty
required(:ids).filled { each(:int?) }

# NOTE: this doesn't apply `each` predicate
required(:ids).filled.each(:int?)

# all array element types are allowed but
# integers must be positive
required(:ids).each { int? > gt?(0) }

# array elements can be nils or positive integers
required(:ids).each { none? | (int? & gt?(0)) }

# multiple predicates for each array value
required(:ids).each(:int?, gteq?: 0)
```

Dry::Validation.Form
--------------------

1. <http://dry-rb.org/gems/dry-validation/forms>

unlike `Dry::Validation.Schema`, `Dry::Validation.Form`:

- symbolizes keys
- coerces values
- coerces empty strings to `nil` (applies to array elements too)

nested data is validated in the same way - even though nested hash
is specified with `schema` macro:

```ruby
required(:quiz).schema do
  required(:question_1).filled(:str?)
end
```

NOTE: `Dry::Validation.Schema` doesn't try to convert string keys
      into atoms and vice versa while `Dry::Validation.Form` performs
      conversion in one direction only: string -> atom.

whitelist keys (like strong parameters)
---------------------------------------

1. <https://github.com/dry-rb/dry-validation/issues/66#issuecomment-195341373>

```ruby
ParamsSchema = Dry::Validation.Schema do
  configure { config.input_processor = :sanitizer }

  required(:foo).maybe
  required(:bar).filled
end

ParamsSchema.call({ foo: 1, bar: 2, baz: 3 })
# => { :foo => 1, :bar => 2 }
```
