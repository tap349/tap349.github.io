---
layout: post
title: monads in Ruby
date: 2017-02-27 00:47:33 +0300
access: public
comments: true
categories: [dry-rb, ruby]
---

notes on using [dry-monads](http://dry-rb.org/gems/dry-monads) and
[dry-matcher](http://dry-rb.org/gems/dry-matcher).

<!-- more -->

definition of monad
-------------------

- <https://www.youtube.com/watch?v=ZhuHCtR3xq8>
- <https://vimeo.com/97344498>
- <https://en.wikipedia.org/wiki/Monad_(functional_programming)>
- <http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html>
- <http://blog.reverberate.org/2015/08/monads-demystified.html>

monad is created by defining:

- type constructors (types)

  in dry-monads `Either` monad is defined using 2 types:
  `Either::Right` and `Either::Left` (they are subclasses of `Either` class -
  but it's just a Ruby-specific implementation detail).

  monadic value (aka lifted value, a value with a context) is an instance of the
  monad's type. when some monad value is mentioned it's monadic value that is
  implied - (`Either` value is monadic value - not plain value wrapped into this
  monad). to lift a value is to wrap plain value in a monad: `Either::Right(5)`.

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

monads in dry-monads
--------------------

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

  - function normalizes url or adds UTM tags to url

- `tee` (`Maybe` and `Either` monads only)

  use `tee` when function is called for side effects only and
  doesn't return anything meaningful that must be passed down the chain:

  - function queues asynchronous tasks (Sidekiq)
  - function creates or updates related models (associations)
    but not operation model itself

### tips

- function should always return `Either` value for uniform processing

  for all monad methods:

  - if calling another service or operation (say, from DI container)
    that returns `Either` value return either that monadic value if it
    wraps what we need for further processing:

    ```ruby
    def _update_site_status model
      result = update_site.(site_id: model.id, status: 'ACTIVE')

      if result.success?
        # return model from operation
        Right(result.value)
      else
        # but still need to generate error message because
        # operation returns model with errors in case of failure
        Left(operation_error_message(update_site, result.value))
      end
    end
    ```

    or new `Either` value that wraps what we need (probably leaving
    original `Left` value intact if it contains comprehensive error
    message with error source):

    ```ruby
    def _create_site_setting model
      result = create_site_setting.(site_id: model.id)

      if result.success?
        # return input model because operation returns another model
        Right(model)
      else
        # generate error message as usual in case of failure
        Left(operation_error_message(create_site_setting, result.value))
      end
    end
    ```

    error handling is organized as follows (for `Create` operation):
    if model is invalid after calling `create` return `Left(model)`
    (model with errors is required for rendering form once again).
    if model is valid call `after_create` that might return either
    `Right(model)` or `Left(error_message)`:

    - if success return wrapped model from the whole operation
    - if failure error is raised with supplied error message

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

- as a consequence of a previous tip always convert `Try` value to `Either` one
  using `Try#to_either` and in particular because:

  - `Try` monad doesn't implement `tee` method
    (if you need to chain on result using `tee` method)
  - `Failure#value` returns `nil` while `Left#value` returns exception
    itself from `Failure#exception` - we need it to generate error message
    (in case of failure this result is eventually returned from the chain)

- when creating/updating model in place it's better to use methods that throw
  exceptions (`create!`/`update!`/etc.) and wrap them in `Try` monad
  (don't forget to convert the latter to `Either` value - see previous tips)
  than to use their counterparts without `!` and check for errors manually
  (returning either `Right(model)` or `Left(model.errors.full_messages)`).
  rationale: exception contains all the necessary validation error messages.

- when creating/updating model association or another related model vice versa
  it's better to use methods that don't throw exceptions (`create`/`update`/etc.)
  and check for errors manually (returning either `Right(model)` or
  `Left(model_error_message(another_model))`) than to use their counterparts
  with `!` and wrap them in `Try` monad (converting to `Either` value in the end).
  rationale: exception doesn't contain information about another model - it's
  necessary to provide our own custom model error message to avoid ambiguity.

- it's recommended to specify expected exceptions when using `Try` monad

- don't call functions that return `Either` value inside `Try` monad blocks

  such functions are not supposed to throw exceptions and
  should not be wrapped in `Try` monad at all.
  moreover their result (`Either` value) will be always wrapped in `Success`
  thus plain value will be double wrapped in 2 monads:
  first `Either`, then `Try` (`Success(Right(model))`).
  in the end this will be unlifted to monadic value - not plain value.

- monadic value can be created either by passing proc or block

  - <http://stackoverflow.com/a/35713455/3632318>
  - <http://www.skorks.com/2013/04/ruby-ampersand-parameter-demystified/>
  - <http://www.iain.nl/going-crazy-with-to_proc>

  `Method` objects can be passed as well (obtained with `Object.method`):

  ```ruby
  # block is given that will be called with `yield` passing
  # current wrapped plain value as argument inside of `bind`
  # (&foo in method call == foo.to_proc and use it as method block)
  Right(model).bind { |model| set_main_mirror(model) } # block
  Right(model).bind(&method(:set_main_mirror)) # method object used as block

  # if no block is given the 1st argument (say, proc or method object)
  # will be called with `call` using remaining arguments to `bind` as
  # its own arguments
  Right(model).bind(method(:set_main_mirror)) # method object
  Right(model).bind(method(:set_main_mirror).to_proc) # proc
  # you can even pass method object that references method in another class:
  Right(model).bind(AnotherClass.method(:set_main_mirror)) # method object

  # in this case `foo` string is not passed as argument to block but rather
  # used as receiver of `upcase` message because of special `Symbol#to_proc`
  # implementation (it's in C now) which is roughly equivalent to:
  #
  # Proc.new { |obj, *args| obj.send(self, *args) }
  #
  # where `obj` is `foo` string, and `self` is `upcase` symbol.
  Right('foo').bind(&:upcase)
  ```

  see `bind` implementation for details:
  [Dry::Monads::RightBiased::Right#bind](https://github.com/dry-rb/dry-monads/blob/master/lib/dry/monads/right_biased.rb#L21).

- pass additional arguments to wrapped function as additional arguments of
  monad method - after the 1st argument of the latter which is either block
  or proc (see previous tip)

  both block and proc inside monad method will receive those additional
  arguments after the 1st argument - unlifted return value from previous
  function call in the pipeline (if it was successful of course).


  ```ruby
  # using method object (or proc - it doesn't matter)

  Right(model).tee(method(:collect_products), force_collect)

  def collect_products model, force_collect
    ...
  end

  # using block

  Right(model).tee(force_collect) do |model, force_collect|
    CollectProductsJob.perform_later(model, force_collect)
  end
  ```

- use `fmap` when you would need pipe operator (which Ruby doesn't have)
  to chain functions as an alternative to
  [chainable_methods](https://github.com/akitaonrails/chainable_methods/):

  ```ruby
  # chainable_methods
  chain_from([1, 3, 2]).filter.sort.unwrap

  # fmap from dry-monads
  Right([1, 2, 3]).fmap(method(:filter)).fmap(method(:sort)).value
  ```

  advantage over `chainable_methods` in this case is that `method(:sort)` will
  return `Method` object that references `sort` method defined in current class
  while calling `sort` in the first example will call `Enumerable#sort` instead.

### example of using dry-monads

```ruby
require 'my_app/inject'

class Site::Create
  include Dry::Monads::Either::Mixin
  include Dry::Monads::Try::Mixin

  include MyApp::Inject['svcs.fetch_main_mirror', 'ops.create_site_setting']

  alias m method

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

    if result.success?
      Right(model)
    else
      Left(operation_error_message(create_site_setting, result.value))
    end
  end

  def collect_products model, force_collect
    return Right(nil) unless model.data_source_SITE?
    CollectProductsJob.perform_later(model, force_collect)

    Right(nil)
  end
end
```

```ruby
class MergeXpaths
  include Dry::Monads::Either::Mixin

  alias :m :method

  def call xpaths
    Right(xpaths)
      .fmap(m(:filter))
      .fmap(m(:sort))
      .value
      .join('|')
  end

  private

  def filter xpaths
    xpaths.select(&:present?).uniq
  end

  def sort xpaths
    xpaths.sort_by(&:size)
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
    if result.failure?
      raise OperationError, error_message(result.value, model)
    end
  end
end
```
