---
layout: post
title: Elixir - Mix
date: 2017-05-21 12:08:46 +0300
access: public
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

see [Phoenix]({% post_url 2016-11-12-phoenix %}) for Phoenix-specific Mix tasks.

tasks can be batched:

```sh
$ mix do deps.get, compile
```

## _mix.exs_

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

## gotchas

- Mix doesn't allow a task to be run twice (just like Rake)

  1. <https://stackoverflow.com/questions/36846041>

  that is why it's impossible to create Mix alias in _mix.exs_
  that runs the same task multiple times with different arguments:

  ```elixir
  defp aliases do
    [
      # works
      "ecto.reset": ["ecto.drop", "ecto.setup"],
      # doesn't work
      "deploy.prod": [
        "edeliver build release",
        "edeliver deploy release production"
      ]
    ]
  end
  ```

  though it's allowed to alternate between 2 tasks, say:

  ```elixir
  defp aliases do
    [
      # works
      "edeliver.all": [
        "edeliver build release",
        "deps.get",
        "edeliver deploy release production"
      ]
    ]
  end
  ```
