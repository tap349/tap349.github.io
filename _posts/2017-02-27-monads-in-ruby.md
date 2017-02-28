---
layout: post
title: monads in Ruby
date: 2017-02-27 00:47:33 +0300
access: public
categories: [dry-rb, ruby]
---

notes on using [dry-monads](http://dry-rb.org/gems/dry-monads/).

<!-- more -->

## definition of monad

- <https://vimeo.com/97344498>
- <https://en.wikipedia.org/wiki/Monad_(functional_programming)>
- <http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html>

monad is created by defining:

- type constructors (types)

  in dry-monads `Either` monad is defined using 2 types:
  `Either::Right` and `Either::Left` (they are subclasses of `Either` class -
  but it's just a Ruby-specific implementation detail).

  to lift a value is to create a monad from a plain value: `Either::Right(5)`.
  lifted value is also a monadic value.

- operations (only the first one is obligatory for all monads):

  - `bind: m a -> (a -> m b) -> m b`

    `bind` function is often aliased as `>>=` (like in Haskell).

    it takes one monadic value (wrapping `a`) and a function transforming `a` into
    another monadic value (wrapping `b`) and returns that another monadic value.

    it allows for a function accepting plain value to have monadic value as input:
    `bind` function either
    (1) unlifts monadic value feeding wrapped plain value to the function
    (when success - monadic value has `Either::Right` type for `Either` monad)
    or (2) immediately returns original monadic value
    (when failure - monadic value has `Either::Left` type for `Either` monad).

  - `fmap: m a -> (a -> b) -> m b`

    it's used for single-track function (term from [ROP](http://fsharpforfunandprofit.com/rop/)) -
    they are supposed to succeed all the time.
    it just lifts the result for you (wraps return value in monad).

  - `tee: m a -> (a -> m b) -> m a` (for dry-monads only)

    in case of dry-monads `tee` function is supposed to take monadic value
    (`Try` monad doesn't implement it - only `Maybe` and `Either` monads do)
    and return monadic value as well - the latter is used only in case of
    failure otherwise input monadic value is returned.

## monads in dry-monads

### monads

3 monads are available in dry-monads: `Maybe`, `Either` and `Try` -
each having 2 type constructors:

- `Maybe`: `Some`/`None`
- `Either`: `Right`/`Left`
- `Try`: `Success`/`Failure`

  use it to call function that:

  - uses some library functions (gems) that might throw exceptions and
    you can't control it in any other way
  - creates and updates model in place with method that throws exception
    (`create!`/`update!`/etc.)

### monad methods

monads have the following methods:

- `bind` (all monads)

  use `bind` when you need to pass function result down the chain:

  - function updates operation model

- `fmap` (all monads)

  use `fmap` when you need to pass function result down the chain
  and nothing can go wrong in that function:

  - function normalizes url or adds utm tags to url

- `tee` (`Maybe` and `Either` monads only)

  use `tee` when function is called for side effects only and
  doesn't return anything meaningful that must be passed down the chain:

  - function queues asynchronous tasks (Sidekiq)
  - function creates or updates related models (associations)
    but not operation model itself

### tips

- always convert `Try` monad to `Either` one using `Try#to_either` because:

  - `Try` monad doesn't implement `tee` method
    (if you need to chain on result using `tee` method)
  - `Try::Failure#value` returns nil while `Either::Left#value` returns exception
    itself from `Try::Failure#exception` - we need it to generate error message
    (in case of failure this result is eventually returned from the chain)

- function should always return `Either` monad for uniform processing:

  - for `bind` return:
    - `Right(model)` or `Left(error_message)` if nothing returns monad
    - result of calling another function if it returns `Either` monad
      (say, some service or operation from DI container)
    - `Try` monad converted to `Either` one if the former is used
      (`Try` monad block must return something meaningful - e.g. model)

  - for `tee` return:
    - dummy `Right(nil)` or `Left(error_message)` if nothing returns monad
      (since we don't care about the result in case of success)
    - result of calling another function if it returns `Either` monad
      (say, some service or operation from DI container)
    - `Try` monad converted to `Either` one if the former is used
      (`Try` monad block can return anything - it won't be used anyway)

- when updating model in place it's better to use methods that throw exceptions
  (`create!`/`update!`/etc.) and wrap them in `Try` monad
  (don't forget to convert it to `Either` monad - see the first tip)
  than to use their counterparts without `!` and check for errors manually
  (returning either `Right(model)` or `Left(model.errors.full_messages)`).
  rationale: exception contains all the necessary validation error messages.

- it's recommended to specify expected exceptions when using `Try` monad

### using dry-monads

- <http://blog.reverberate.org/2015/08/monads-demystified.html>

here I use monad and monadic value interchangeably though it's not the same
(monadic value is an instance of the monad's type):

```ruby
require 'my_app/inject'

class Site::Create < CreateBase
  include Dry::Monads::Either::Mixin
  include Dry::Monads::Try::Mixin

  include MyApp::Inject[
    'svcs.fetch_main_mirror',
    'ops.create_site_setting'
  ]

  alias :m :method

  MAIN_MIRROR_NOT_FETCHED = 'error fetching main mirror'
  DB_SITE_SETTING_NOT_CREATED = 'site setting not created'

  # pass procs (Method objects) instead of blocks to monad methods
  #
  # pass additional function arguments after the proc argument
  # (the first function argument is unlifted return value from previous
  # function call in the pipeline - if it was successful of course)
  def after_create model, force_collect
    Right(model)
      .tee(m(:update_email))
      .tee(m(:send_email))
      .bind(m(:set_main_mirror))
      .bind(m(:_create_site_setting))
      .fmap(m(:collect_products), force_collect)
  end

  def update_email! model
    Try { model.update! email: model.user.email }.to_either
  end

  def send_email model
    Right SiteMailer.site_created(model).deliver_later
  end

  # - generate custom error message if fetching main mirror failed:
  #   service always returns url (original url in case of failure)
  # - return Either monad explicitly after updating model
  #   (wraps model in case of success and error message in case of failure)
  def set_main_mirror model
    result = fetch_main_mirror.(model.domain)
    return Left(MAIN_MIRROR_NOT_FETCHED) if result.failure?

    model.update(domain: URL.host(result.value))
    model.errors.none? ? Right(model) : Left(model.errors.full_messages)
  end

  # TODO: add site setting error messages if it's not created
  def _create_site_setting model
    result = create_site_setting.(site_id: model.id)
    result.success? ? Right(model) : Left(DB_SITE_SETTING_NOT_CREATED)
  end

  # we don't care about the result here and thus can use just fmap that
  # lifts the result for us (that is wraps any return value into Right)
  def collect_products model, force_collect
    return unless model.data_source_SITE?
    Diffbot::AnalyzeJob.perform_assured(model.site_setting, force_collect)
  end
end
```

### using dry-matcher to match on result

```ruby
require 'dry/matcher/either_matcher'

class OperationBase
  def raise_on_failure! result, model
    # works both with Either and Try monads
    Dry::Matcher::EitherMatcher.(result) do |m|
      m.failure do |value|
        raise OperationError, error_message(value, model)
      end
    end
  end
end
```

in this simple case it's possible to avoid using matcher at all:

```ruby
class OperationBase
  def raise_on_failure! result, model
    raise OperationError, error_message(value, model) if result.failure?
  end
end
```
