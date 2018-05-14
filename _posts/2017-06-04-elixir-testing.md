---
layout: post
title: Elixir - Testing
date: 2017-06-04 14:25:16 +0300
access: public
comments: true
categories: [elixir, testing]
---

<!-- more -->

```sh
$ mix test
```

notes
-----

### mocks

1. <http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/>

> Passing the dependency as argument is much simpler and should be
> preferred over relying on configuration files and Application.get_env/3.
> When not possible, the configuration system is a good fallback.

> Another way to think about mocks is to treat them as nouns.
> You shouldn’t mock an API (verb), instead you create a mock
> (noun) that implements a given API.
>
> When you use mock as a verb, you are changing something that
> already exists, and often those changes are global.
> When you use mock as a noun, you need to create something new,
> and by definition it cannot be the SomeDependency module because
> it already exists.

<https://github.com/plataformatec/mox/>:

when using `Mox.stub` it's almost the same as just defining mock module
except that we don't do it in our codebase polluting it with mock modules
(usually Mox mocks are stubbed inside tests or in _test_helper.exs_).

### setup vs. setup_all

`context` variable in `setup_all` doesn't have `test` key:

```elixir
%{case: MyApp.FooTest, module: MyApp.FooTest}
```

this makes sense since `setup_all` is run before the whole suite -
not before each test.

style guide
-----------

1. <https://groups.google.com/forum/#!topic/elixir-ecto/BKpLf092dWs>

<https://groups.google.com/d/msg/elixir-ecto/BKpLf092dWs/VaCvfZpEBQAJ>:

> * test/models/user_test.exs - you will test your changeset and others.
> Again, only data transformation, tests can be "async: true" because it
> doesn't talk to the database.
>
> * test/models/user_repo_test.exs - you will test anything the model
> returns that needs the repository to be tested, like complex queries.
> Here it makes no sense to mock because you *need* the repo, testing a
> complex query against a fake repo makes no purpose. Also, since it
> depends on the Repo, tests cannot be concurrent (until Ecto 1.1 where
> we solve this problem)
>
> * test/views/user_test.exs - you will test how your views are rendered.
> Again, it is only data transformation, so tests can be "async: true"
> because it doesn't talk to the database.
>
> * test/controllers/user_controller_test.exs - integration layer. You'll
> test a simple pass through the different layers and the expected result.
> Important: you are not going to test "models" and "views" combinations here.
> For example, you won't test that when you POST attributes A and B, field C
> will be added to the model. That's is going to be in the model test. In the
> same way you are not going to test how users with different attributes are
> rendered. That's in the view test.

tips
----

### add _test/support/_ to compilation paths

1. <https://github.com/thoughtbot/ex_machina#install-in-just-the-test-environment-for-non-phoenix-projects>
2. <https://elixirforum.com/t/load-module-during-test/7400/2>

_mix.exs_:

```elixir
def project do
  [
    app: :my_app,
    # ...
    elixirc_paths: elixirc_paths(Mix.env)
  ]
end

defp elixirc_paths(:test), do: ["lib", "test/support"]
defp elixirc_paths(_), do: ["lib"]
```

***UPDATE***

when generating new Phoenix projects, _test/support/_ is added to
compilation paths by default.

### use MyApp.DataCase in tests with access to data layer

1. <https://hexdocs.pm/phoenix/testing_schemas.html#test-driving-a-changeset>

<http://whatdidilearn.info/2018/04/01/testing-phoenix-models-and-controllers.html>:

> By default, Phoenix uses the Ecto.Adapters.SQL.Sandbox module. Which
> basically wraps every test within a transaction in order to rollback
> it after a test is finished. That helps to keep the test database clean.
>
> "Testing Schemas" guide does not recommend us to use async: true option
> if we are going to interact with the database:
>
> > Note: We should not tag any schema case that interacts with a database
> > as :async. This may cause erratic test results and possibly even deadlocks.
>
> Although, Ecto.Adapters.SQL.Sandbox also contains a note about that:
>
> > While both PostgreSQL and MySQL support SQL Sandbox, only PostgreSQL
> supports concurrent tests while running the SQL Sandbox. Therefore, do
> not run concurrent tests with MySQL as you may run into deadlocks due
> to its transaction implementation.
>
> We are using PostgreSQL for that project. So I am going to enable that
> option at my own peril.

_test/support/data_case.ex_:

```diff
- use ExUnit.CaseTemplate
+ use ExUnit.CaseTemplate, async: true
```

or else it's possible to do it inside specific test module:

```diff
- use Sithex.DataCase
+ use Sithex.DataCase, async: true
```

### (how to) run specific tests (same as `focus` in RSpec)

- provide test path and line number to `mix test`

  ```sh
  $ mix test test/foo.exs:14
  ```

- use custom tag

  _test/foo.exs_:

  ```elixir
  @tag :wip
  test "run this test only" do
    assert true
  end
  ```

  ```sh
  $ mix test --only wip
  ```

### (how to) show detailed information (same as `--format documentation` in RSpec)

```sh
$ mix test --trace
```

`--trace` option also sets timeout to infinity - useful when using `IEx.pry`.

### (how to) run tests synchronously

`async: true` has no effect - useful in case of race conditions.

```sh
$ mix test --trace
```

### (how to) use IEx.pry in tests

1. <https://stackoverflow.com/a/34863997/3632318>

to make pry work in tests run `mix test` in IEx session:

```sh
$ iex -S mix test
```

in my case `iex` is aliased to `iex -S mix` so type just `iex test`.

NOTE: test will time out after 60000ms by default:

```
** (ExUnit.TimeoutError) test timed out after 60000ms. You can change the timeout:

  1. per test by setting "@tag timeout: x"
  2. per case by setting "@moduletag timeout: x"
  3. globally via "ExUnit.start(timeout: x)" configuration
  4. or set it to infinity per run by calling "mix test --trace"
     (useful when using IEx.pry)

Timeouts are given as integers in milliseconds.
```

### (how to) use Logger in tests

1. <https://stackoverflow.com/a/36349341/3632318>

_config/test.exs_:

```elixir
- config :logger, level: :warn
+ config :logger, level: :info
```

this will print all info messages in tests:

```elixir
Logger.info("foo")
```

### (how to) pass params in controller tests

1. <https://medium.com/@lasseebert/test-driving-a-phoenix-endpoint-part-i-b53e300c1a0a>

params can be specified as either keyword list or map:

```elixir
conn = get(conn, webhook_path(conn, :show), %{"hub.mode" => "subscribe"})
# or
conn = get(conn, webhook_path(conn, :show), ["hub.mode": "subscribe"])
```
