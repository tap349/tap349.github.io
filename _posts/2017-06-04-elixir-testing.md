---
layout: post
title: Elixir - Testing
date: 2017-06-04 14:25:16 +0300
access: public
comments: true
categories: [elixir, testing]
---

<!-- more -->

* TOC
{:toc}
<hr>

<dl>
  <dt>test case</dt>
  <dd>test module</dd>
</dl>

<hr>

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

### external APIs

> <https://github.com/plataformatec/mox/issues/9#issuecomment-423607089>
>
> The API you are going to mock is not HTTPoison.get! because that's not your
> domain. You are going to mock something like MyApp.TwitterClient.get_tweets
> which internally calls HTTPoison.get! (or whatever else).

so it's necessary to mock application boundaries in tests (if they reach these
boundaries of course), API clients are your application boundaries => mock API
clients (not HTTP client) - say, `Lain.API.Google.Drive` rather than low-level
`Lain.API.HTTPClient` which is used under the hood of the former.

at the same time it might be necessary to test API clients themselves - there
are several ways to do it:

- *[RECOMMENDED]* hit real endpoint

  > <http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts>
  >
  > Personally, I would test MyApp.Twitter.HTTP by directly reaching the
  > Twitter API. We would run those tests only when needed in development
  > and configure them as necessary in our build system. The @tag system
  > in ExUnit provides conveniences to help us with that.

  ```elixir
  defmodule Lain.API.Google.DriveTest do
    use ExUnit.Case, async: true

    @moduletag :api
  end
  ```

  don't run tests tagged `api` by default:

  ```elixir
  # test/test_helper.exs

  ExUnit.configure(exclude: [:api])
  ```

  to include tests tagged `api`:

  ```sh
  $ mix test --include api
  ```

- emulate external API with dummy webserver

  1. <https://pspdfkit.com/blog/2016/testing-http-apis-in-elixir>

  use [bypass](https://github.com/pspdfkit-labs/bypass), for example.

- use HTTP client mock

  use [Mox](https://github.com/plataformatec/mox), for example.

  HTTP client mock must be used for testing API clients only - use API client
  mocks everywhere else.

- record and replay HTTP interactions

  use [ExVCR](https://github.com/parroty/exvcr) - it's the only option AFAIK.

### setup vs. setup_all

`context` variable in `setup_all` doesn't have `test` key:

```elixir
%{case: MyApp.FooTest, module: MyApp.FooTest}
```

this makes sense since `setup_all` is run before the whole suite -
not before each test.

### about `async: true`

> <https://github.com/elixir-lang/elixir/issues/3580#issuecomment-130860923>
>
> Tests inside a test case are always run serially but whole cases can
> run in parallel with other cases with `async: true`.

=> test cases without `async: true` are not run in parallel with test cases
with `async: true`.

### [Mox] Mox.expect/4 vs. Mox.stub/3

1. <https://martinfowler.com/articles/mocksArentStubs.html>
2. <https://blog.carbonfive.com/2018/01/16/functional-mocks-with-mox-in-elixir>

> <https://stackoverflow.com/a/3459407/3632318>
>
> A Mock is just testing behaviour, making sure certain methods are called.
> A Stub is a testable version (per se) of a particular object.

`Mox.expect/4` sets expectations and allows to verify them (that some function
in a mock is called exactly n times).

`Mox.stub/3` doesn't set expectations - stubs are never verified (that is it's
possible that stub will never be invoked at all).

as a rule of thumb I use:

- `Mox.expect/4` when mock is injected into tested module directly
- `Mox.stub/3` when mock is used by tested module via other modules
  (say, in controller or integration tests)

> <https://hexdocs.pm/mox/Mox.html#stub/3>
>
> stub is invoked only after all expectations are fulfilled

=> expectations have precedence over stubs: the latter will be used as a
fallback when no expectations are set.

### [Mox] Mox.stub_with/2

it's possible to stub all functions in a mock at once by defining a separate
stub module and passing it to `Mox.stub_with/2`:

```elixir
# test/support/stubs/lain/api/api_stub.ex

defmodule Lain.APIStub do
  @behaviour Lain.API.Behaviour

  @impl true
  def list_page_conversations(_cursor) do
    # return test data
  end
end
```

```elixir
# lib/lain/chat/conversation/operations/import.ex

setup do
  Mox.stub_with(Lain.APIMock, Lain.APIStub)
  :ok
end
```

> <https://github.com/plataformatec/mox/issues/41#issuecomment-412998957>
>
> It (stub_with/2) needs to be inside setup, since setup_all runs in a separate
> process.

=> you can't place this line in _test/test\_helper.exs_ and rest assured that
all calls to mock are now stubbed by default. the problem is that all tests
are run in separate processes (regardless of `async` option value) while mock
is stubbed in current process only - in this case in the process in which Mix
loads _test/test\_helper.exs_ before executing tests (for some unknown reason
this process always has `#PID<0.91.0>` on my machine).

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

### test names

comparison with RSpec:

- `describe` (ExUnit) == `describe` or `context` (RSpec)
- `test` (ExUnit) == `it` + `context` (RSpec)

`test` message variants (`human message` → `test message`):

- `test doing smth` → `do smth`

  it's like you continue the phrase starting with `test` and
  replace gerund with plain infinitive in the end.

  ```elixir
  # test getting user by email =>
  test "get user by email" do
  end
  ```

- `test that smb/smth is/does smb/smth` → `smb/smth is/does smb/smth`

  ```elixir
  # test that user is admin now =>
  test "user is admin now" do
  end

  # test that user has become admin =>
  test "user has become admin" do
  end
  ```

- `test that function does smth` → `does smth`

  this naming convention is adopted in RSpec since tests are starting
  with `it` there.

  ```elixir
  # it converts string to integer
  test "converts string to integer" do
  end
  ```

  this variant is commonly used when test is wrapped in `describe` block:

  ```elixir
  describe "to_integer/2" do
    # to_integer/2 converts string to integer
    test "converts string to integer" do
    end
  end
  ```

### assertions

place result of actual call (result of calling the function under test)
on the LHS when comparing it with expected expression:

```elixir
assert actual == expected
```

except for the cases when it's necessary to use pattern matching instead
of strict comparison:

```elixir
assert %{name: "foo"} = MyModule.call()
```

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
> "Testing Schemas" guide does not recommend us to use `async: true` option
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

1. [Elixir - Debugging]({% post_url 2017-07-30-elixir-debugging %})
2. <https://stackoverflow.com/a/34863997/3632318>

to make pry work in tests run `mix test` inside `iex` session:

```sh
$ alias iex='iex -S mix'
$ iex test --trace
```

test will time out after 60000 ms by default unless `--trace` option is used:

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

### (how to) mock application configuration in tests

1. <https://elixirforum.com/t/using-application-get-env-application-put-env-in-exunit-tests/8019>

sometimes it might be necessary to mock application configuration - say,
to set some configuration paramater to invalid value.

general recommendations:

- read configuration parameter at runtime (not via module attribute) so
  that it's possible to mock it in tests
- don't use `async: true` to prevent other tests from using mocked value
  accidentaly
- restore environment after the test using `ExUnit.Callback.on_exit/2`

client module:

```elixir
defmodule MyApp.API.AccessToken do
  # ...

  defp refresh_token do
    Application.get_env(:my_app, :api)[:refresh_token]
  end
end
```

test module:

```elixir
defmodule MyApp.API.AccessTokenTest do
  use ExUnit.Case, async: false
  # ...

  test "returns error when invalid request (invalid refresh token)" do
    api_config = Application.get_env(:my_app, :api)
    on_exit(fn -> Application.put_env(:my_app, :api, api_config) end)

    new_api_config =
      :my_app
      |> Application.get_env(:api)
      |> put_in([:refresh_token], "foo")

    Application.put_env(:my_app, :api, new_api_config)

    {:error, message} = MyApp.API.AccessToken.get()
    assert "Error refreshing access token: 400: " <> _ = message
  end
end
```

### (how to) mock current environment in tests

1. [Elixir - Tips](%{ post_url 2017-07-14-elixir-tips })

see the tip from [Elixir - Tips](%{ post_url 2017-07-14-elixir-tips }) on
how to get current environment at runtime.

if you need to test both production and non-production code branches, read
read `:env` configuration paramater dynamically (not via module attribute):

```diff
  defmodule MyApp.Foo do
-   @env Application.get_env(:my_app, :env)
-
    def call do
-     if @env == :prod do
+     if Application.get_env(:my_app, :env) == :prod do
        # production code
      else
        # non-production code
      end
    end
  end
```

then mock `:env` configuration parameter as described in the tip above.

### (how to) test without starting your application and certain deps

1. <https://virviil.github.io/2016/10/26/elixir-testing-without-starting-supervision-tree>
2. <https://elixirforum.com/t/ecto-starting-in-test-environment/1205/6>
3. <https://hexdocs.pm/ecto/Ecto.Repo.html#c:start_link/1>
4. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#c:start_link/0>

it's meant to make `mix test` run faster thanks to not starting certain
deps which take a lot of time to start (say, `quantum`).

don't start your application and deps (which are applications too) when
running tests:

```
$ mix help test
...
--no-start - does not start applications after compilation
```

```diff
# mix.exs
  defp aliases do
    [
      # ...
-     test: ["ecto.create --quiet", "ecto.migrate", "test"],
+     test: ["ecto.create --quiet", "ecto.migrate", "test --no-start"],
      # ...
    ]
  end
```

start certain deps along with `Repo` and `Endpoint` supervisors manually:

```elixir
# test/test_helper.exs

# -------------------------------------------------------------------
# start seleted applications only
# -------------------------------------------------------------------

# load application first to get access to its spec:
#
# > https://hexdocs.pm/elixir/Application.html#spec/1
# >
# > Returns nil if the application is not loaded.
Application.load(:lain)

not_started_apps = ~w(
  distillery
  observer_cli
  phoenix_live_reload
  quantum
)a

for app <- Application.spec(:lain, :applications),
    app not in not_started_apps do
  Application.ensure_all_started(app)
end

# -------------------------------------------------------------------
# start required supervisors from Lain.Application manually
# -------------------------------------------------------------------

# for Phoenix application only
#
# > https://elixirforum.com/t/ecto-starting-in-test-environment/1205/6
# >
# > Calling Repo.start_link in test_helper.exs is the correct approach.
Lain.Repo.start_link()
LainWeb.Endpoint.start_link()
# https://hexdocs.pm/elixir/Task.Supervisor.html#start_link/1
Task.Supervisor.start_link(name: Lain.TaskSupervisor)

# -------------------------------------------------------------------
# standard configuration
# -------------------------------------------------------------------

ExUnit.start()

Ecto.Adapters.SQL.Sandbox.mode(Lain.Repo, :manual)
```

### (how to) print query SQL in logs

1. <https://stackoverflow.com/questions/42236123>

set log level to `debug` in test environment:

```diff
  # config/test.exs

- config :logger, level: :warn
+ config :logger, level: :debug
```

### (how to) test GenServers and modules using them

- processing asynchronous messages

  1. <https://blog.eq8.eu/til/exunit-wait-for-genserver-cast.html>

  one problem with testing GenServers is that we need to make sure some
  asynchronous message is processed - such message might be sent either
  from inside GenServer itself (for example, via `Process.send_after/3`
  when performing periodic work) or from outside via `GenServer.cast/2`.

  solution is to send this message and wait for it to be processed using
  `:sys.get_state/1` - in fact it's possible to use any synchronous call
  to this GenServer since all messages are processed serially one by one
  and once synchronous call returns all previous messages must have been
  already processed as well:

  ```elixir
  setup do
    # also make sure `async: true` is not set in corresponding test
    # case since the same name (module name) would be used for all
    # workers (workers are named GenServers and their names must be
    # unique - attempt to start GenServer with the same name will
    # just return {:error, {:already_started, pid}})
    {:ok, child_pid} = Lain.FB.Label.Sync.start_link([])
    # send :work message to sync worker manually
    send(child_pid, :work)
    # wait till :work message is processed
    :synced = :sys.get_state(child_pid)
    # => worker is synced now

    :ok
  end
  ```

- [Mox] allow GenServer to use expectations and stubs from test process

  1. <https://hexdocs.pm/mox/Mox.html#module-multi-process-collaboration>

  it's safe to use `Mox.set_mox_global/1` only in test cases without
  `async: true`, it's always safe to use explicit allowances:

  ```elixir
  setup do
    # start GenServer and save its PID into child_pid

    Lain.APIMock
    # use Lain.APIStub in Lain.FB.Label.Sync.sync_now() - mock is not
    # stubbed in the tests as each test is run in a separate process
    |> Mox.stub_with(Lain.APIStub)
    # allow child process (GenServer process) to use expectations
    # and stubs defined for mock in parent process (test process)
    |> Mox.allow(self(), child_pid)
  end
  ```

if internal message (say, the first periodic message) is used to initialize
GenServer state it's possible to set up everything in _test/test_helper.exs_:

```elixir
# test/test_helper.exs

# -------------------------------------------------------------------
# - start required workers from Lain.Application manually
# - allow them to use stubs defined in current test process
# - wait till they are ready
# -------------------------------------------------------------------

{:ok, child_pid} = Lain.FB.Label.Sync.start_link([])

Lain.APIMock
|> Mox.stub_with(Lain.APIStub)
|> Mox.allow(self(), child_pid)

# initialize state manually by sending the first periodic message
send(child_pid, :work)
:synced = :sys.get_state(child_pid)

# or else by making a synchronous call - if specified message is
# handled by handle_call/3 callback of course
:ok = GenServer.call(Lain.FB.Label.Sync, :sync_now)
```

### (how to) suppress logs in tests

1. <https://minhajuddin.com/2019/02/01/pearls-of-elixir-interesting-patterns-from-popular-elixir-packages/>

```diff
  defmodule Francesca.FB.Ad.Operations.CollectDailyAdStatsTest do
    use Francesca.DataCase
    import Mox

+   @moduletag :capture_log

    # ...
  end
```
