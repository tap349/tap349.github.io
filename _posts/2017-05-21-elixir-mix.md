---
layout: post
title: Elixir - Mix
date: 2017-05-21 12:08:46 +0300
access: public
comments: true
categories: [elixir, mix]
---

<!-- more -->

* TOC
{:toc}
<hr>

> Mix assumes that we are in the development environment unless we tell it
> otherwise with MIX_ENV=<another_environment> mix some_task.

## common tasks

- `mix compile`

  compiles the whole project including dependencies.

- `mix deps.compile [<package>]`

  compiles project dependencies only (say, when you need to
  recompile dependencies after editing their source code manually).

- `mix deps.get` = `bundle install`

  gets all missing and out of date dependencies
  (updates existing ones up to the version specified in _mix.lock_).

- `mix deps.clean --unused --unlock` = `bundle install`

  removes dependencies which are no longer listed in _mix.exs_
  (`--unused`) and updates _mix.lock_ (`--unlock`).

- `mix deps.update --all` = `bundle update`
- `mix deps.update <package>` = `bundle update <gem>`

  updates dependency and writes updated version to _mix.lock_.

tasks can also be batched:

```sh
$ mix do deps.get, compile
```

## Phoenix tasks

- `mix phx.server` = `rails server`
- `mix phx.routes` = `rake routes`
- `mix phx.gen.secret` = `rake secret`

## mix.exs

1. <http://blog.plataformatec.com.br/2016/07/understanding-deps-and-applications-in-your-mixfile/>

- application name

  ```elixir
  def project do
    [
      app: :neko,
      ...
    ]
  end
  ```

  - stored in _neko.app_ (created when project is compiled)
  - used to start/stop application manually in IEx

- application inference

  1. <http://elixir-lang.org/blog/2017/01/05/elixir-v1-4-0-released/#application-inference>

  deps should be no longer listed in applications list explicitly:

  ```elixir
  def application do
    [
      # this line is longer necessary - all dependencies
      # are added to `applications` by default in Elixir 1.4
      #applications: [:httpoison],
      # specify only extra applications from Erlang/Elixir
      extra_applications: [:logger],
      mod: {Neko.Application, []}
    ]
  end

  defp deps do
    [
      {:httpoison, "~> 0.11.1"}
    ]
  end
  ```

## troubleshooting

### Mix doesn't allow to run the same task twice in alias

Rake has the same behaviour - task must be reenabled explicitly
before it can be run again.

thus it's impossible to create alias in _mix.exs_ for the list of tasks
in which the same task is run multiple times with different arguments:

```elixir
defp aliases do
  [
    # works
    "ecto.reset": ["ecto.drop", "ecto.setup"],
    # doesn't work (the 2nd task is not run)
    "deploy.prod": [
      "edeliver build release",
      "edeliver deploy release production"
    ]
  ]
end
```

alternating between 2 tasks doesn't reenable the 1st task:

```elixir
defp aliases do
  [
    "edeliver.all": [
      "edeliver build release",
      "deps.get",
      # edeliver task is not reenabled
      "edeliver deploy release production"
    ]
  ]
end
```

**solution**

1. <https://stackoverflow.com/questions/36846041>
2. <https://hexdocs.pm/mix/Mix.Task.html#rerun/2>
3. <https://hexdocs.pm/mix/Mix.html#module-aliases>
4. <https://github.com/elixir-lang/elixir/blob/master/lib/mix/test/mix/task_test.exs#L138>

it's possible to pass function instead of a string with task name and
task arguments - that function would need to use `Mix.Task.rerun/2` to
run the same task multiple times (it reenables the task before running it):

```elixir
defp aliases do
  [
    "deploy.stage": [
      &deploy_stage/1,
      "cmd ssh devops@billing sudo systemctl restart billing_stage"
    ]
  ]
end

defp deploy_stage(_) do
  Mix.shell.info("[billing staging]")
  Mix.Task.run(:edeliver, ["update", "staging", "--mix-env=stage"])
  Mix.Task.rerun(:edeliver, ["migrate", "staging"])
end
```

for some reason it's required to define function with arity of 1 -
otherwise Mix complains:

```
(FunctionClauseError) no function clause matching in Mix.Task.run_alias/3
```
