---
layout: post
title: monads in Ruby
date: 2017-02-27 00:47:33 +0300
access: public
categories: [dry-rb, ruby]
---

notes on using [dry-monads](http://dry-rb.org/gems/dry-monads) and
[dry-matcher](http://dry-rb.org/gems/dry-matcher).

<!-- more -->

## definition of monad

- <https://vimeo.com/97344498>
- <https://en.wikipedia.org/wiki/Monad_(functional_programming)>
- <http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html>
- <http://blog.reverberate.org/2015/08/monads-demystified.html>

monad is created by defining:

- type constructors (types)

  in dry-monads `Either` monad is defined using 2 types:
  `Either::Right` and `Either::Left` (they are subclasses of `Either` class -
  but it's just a Ruby-specific implementation detail).

  monadic value (aka lifted value) is an instance of the monad's type.
  when some monad value is mentioned it's monadic value that is implied -
  (`Either` value is monadic value - not plain value wrapped into this monad).
  to lift a value is to wrap plain value in monad: `Either::Right(5)`.

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

    it's used for single-track function
    (term from [ROP](http://fsharpforfunandprofit.com/rop/)) -
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
each having 2 types (subclasses):

- `Maybe`: `Maybe::Some`/`Maybe::None`
- `Either`: `Either::Right`/`Either::Left`

  `Either::Right` and `Either::Left` will be abbreviated as
  `Right` and `Left` accordingly in this article
  (this is how they are available after including corresponding mixin).

- `Try`: `Try::Success`/`Try::Failure`

  `Try::Success` and `Try::Failure` will be abbreviated as
  `Success` and `Failure` accordingly in this article
  (this is how they are available after including corresponding mixin).

  use it to call function that might throw exception. e.g. when function:

  - uses external library (gem)
  - creates or updates model with bang methods (`create!`/`update!`/etc.)

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

1) always convert `Try` value to `Either` one using `Try#to_either` because:

  - `Try` monad doesn't implement `tee` method
    (if you need to chain on result using `tee` method)
  - `Failure#value` returns nil while `Left#value` returns exception
    itself from `Failure#exception` - we need it to generate error message
    (in case of failure this result is eventually returned from the chain)

2) function should always return `Either` value for uniform processing

  for all monad methods:

  - if calling another service or operation (say, from DI container)
    that returns `Either` value return either that monadic value if it
    wraps what we need for further processing:

    ```ruby
    def _update_site_status model
      update_site.(site_id: model.id, status: 'ACTIVE')
    end
    ```

    or new `Either` value that wraps what we need
    (probably leaving original `Left` value intact if it
    contains comprehensive error message with error source):

    ```ruby
    def _create_site_setting model
      result = create_site_setting.(site_id: model.id)
      result.success? ? Right(model) : result
    end
    ```

  for `bind` return:

  - `Right(model)` or `Left(error_message)` if nothing else returns monadic value
  - `Try` value converted to `Either` one if the former is used
    (`Try` monad block must return something meaningful - e.g. model)

  for `fmap` return:

  - plain value which is supposed to be passed down the chain

  for `tee` return:

  - `Right(nil)` or `Left(error_message)` if nothing else returns monadic value
    (since we don't care about the result in case of success)
  - `Try` value converted to `Either` one if the former is used
    (`Try` monad block can return anything - it won't be used anyway)

3) when updating model in place it's better to use methods that throw exceptions
  (`create!`/`update!`/etc.) and wrap them in `Try` monad
  (don't forget to convert the latter to `Either` value - see previous tips)
  than to use their counterparts without `!` and check for errors manually
  (returning either `Right(model)` or `Left(model.errors.full_messages)`).
  rationale: exception contains all the necessary validation error messages.

4) it's recommended to specify expected exceptions when using `Try` monad

5) don't call functions that return `Either` value inside `Try` monad blocks

  such functions are not supposed to throw exceptions and should not
  be handled using `Try` monad at all.

  moreover their result (`Either` value) will be always wrapped in `Success`:
  `Success(Right(model)).to_either => Right(Right(model))`

- monadic value can be created either by passing method proc or block

  `Method` objects can be passed as well (obtained with `Object.method`) -
  I guess anything that responds to `call` will do:

  ```ruby
  Right(model).bind { |model| set_main_mirror(model) } # block
  Right(model).bind(method(:set_main_mirror)) # method object
  Right(model).bind(&method(:set_main_mirror)) # proc
  ```

6) pass additional function arguments after the proc argument
  (the first function argument is unlifted return value from previous
  function call in the pipeline - if it was successful of course)

  ```ruby
  Right(model).tee(method(:collect_products), force_collect)

  def collect_products model, force_collect
    ...
  end
  ```

  IDK how to pass additional arguments when using blocks.

### example of using dry-monads

```ruby
require 'my_app/inject'

class Site::Create
  include Dry::Monads::Either::Mixin
  include Dry::Monads::Try::Mixin

  include MyApp::Inject['svcs.fetch_main_mirror', 'ops.create_site_setting']

  alias :m :method

  MAIN_MIRROR_NOT_FETCHED = 'error fetching main mirror'

  def after_create model, force_collect
    Right(model)
      .bind(m(:set_main_mirror))
      .bind(m(:_create_site_setting))
      .tee(m(:collect_products), force_collect)
  end

  def set_main_mirror model
    result = fetch_main_mirror.(model.domain)
    return Left(MAIN_MIRROR_NOT_FETCHED) if result.failure?

    Try do
      model.update!(domain: URL.host(result.value))
      model
    end.to_either
  end

  def _create_site_setting model
    result = create_site_setting.(site_id: model.id)
    result.success? ? Right(model) : result
  end

  def collect_products model
    return Right(nil) unless model.data_source_SITE?
    Diffbot::AnalyzeJob.perform_assured(model.site_setting)

    Right(nil)
  end
end
```

### example of using dry-matcher to match on result

```ruby
require 'dry/matcher/either_matcher'

class OperationBase
  def raise_on_failure! result, model
    # works both with Either and Try values
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
