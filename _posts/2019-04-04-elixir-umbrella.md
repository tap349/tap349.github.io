---
layout: post
title: Elixir - Umbrella
date: 2019-04-04 21:51:44 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

notes
-----

### rationale

1. <https://elixirforum.com/t/how-do-you-use-umbrella-apps-and-why/14850>

> <https://elixirforum.com/t/how-do-you-use-umbrella-apps-and-why/14850/8>
>
> My team is using it as monorepo. Some apps are pretty independent so they
> might be in different repo - but for easier workflow we merge then into one
> umbrella project.

> <https://news.ycombinator.com/item?id=17117155>
>
> Basically you could split up your apps and hack on them independently (but
> actually have a good way to share dependencies if needed) and at deploy time
> you could choose to deploy all of them, or just the ones you want.

### you can't configure umbrella application

that is you cannot add any options to umbrella application configuration:

```elixir
# config/config.exs

config :umbrella_app, foo: 123
```

or else you'll get this warning when compiling the project:

```
You have configured application :umbrella_app in your configuration file,
but the application is not available.

This usually means one of:

  1. You have not added the application as a dependency in a mix.exs file.

  2. You are configuring an application that does not really exist.

Please ensure :alice exists or remove the configuration.
```

I guess this is because umbrella application is just a container of other
applications but not a proper application itself - it has no `app` key in
its project configuration unlike child applications (`project/0` function
inside _mix.exs_).

tips
----

### todo list when adding new child application

- Ansible: create _prod.secret.exs_ (umbrella application role)
- Distillery: add to release (_rel/config.exs_)
- Distillery: add repo (`ReleaseTasks` module)
- Bootleg: symlink _prod.secret.exs_ (_config/deploy/production.exs_)
- CircleCI: create _test.secret.exs_ for both stages (_.circleci/config.yml_)

configuration
-------------

### versioning

it makes a lot of sense to come up with some versioning scheme in umbrella
application (it has none by default) - this version can be later used in a
bunch of different configs (which is very handy):

```elixir
# config/appsignal.exs

config :appsignal, :config,
  active: true,
  name: "UmbrellaApp",
  revision: Mix.Project.config()[:version],
  # ...
```

```elixir
# rel/config.exs

release :umbrella_app do
  set(version: Mix.Project.config()[:version])
  # ...
end
```

```elixir
# config/deploy.exs

config :app, :alice
config :version, Mix.Project.config()[:version]
# ...
```

either project version itself or Git revision can be used for this purpose:

- project version (semantic versioning)

  project version option is not added by default since umbrella application
  is not a normal Elixir application (as discussed in `notes` section):

  ```elixir
  # mix.exs

  def project do
    [
      version: "0.1.0",
      # ...
    ]
  end
  ```

- Git revision

  ```elixir
  # mix.exs

  def project do
    [
      version: "0.1.0",
      git_revision: git_revision(),
      # ...
    ]
  end

  # https://stackoverflow.com/a/5694416/3632318
  defp git_revision do
    'git rev-parse --short HEAD'
    |> :os.cmd()
    |> List.to_string()
    |> String.trim()
  end
  ```

  when using Git revision, it's possible to have project version as well
  (say, if you also need semantic versioning) - it just won't be used in
  aforementioned configs.

though it's possible to bump versions of child applications as usual (say, if
version of child application is incremented, version of umbrella application
is incremented as well but not vice versa), I think it's way easier in terms
of maintanence not to bump their versions at all (and reset to default value -
say, `0.1.0`).

### AppSignal

1. <https://docs.appsignal.com/elixir/installation.html#installing-the-package>
2. <https://github.com/jeffkreeftmeijer/appsignal-umbrella-example>

deployment
----------

### Distillery

1. <https://hexdocs.pm/distillery/introduction/umbrella_projects.html>

> <https://hackernoon.com/mastering-elixir-releases-with-distillery-a-pretty-complete-guide-497546f298bc#4eae>
>
> 1. Include the names of your child applications in the applications list
> of the Release configuration.
>
> 2. You can choose to take the Release version number from any of the child
> applications by using current_version(:my_child_app).
>
> As mentioned before, it’s also possible to define more than one Release in
> your rel/config.exs and thus specify multiple Release profiles. This can be
> useful if you want to have Releases that only include certain child
> applications from your umbrella project.

### Bootleg

it's necessary to set version manually when deploying umbrella application
(see `versioning` section above):

```elixir
# config/deploy.exs

config :app, :alice
config :version, Mix.Project.config()[:version]
```

or else you'll get this error:

```
$ mix bootleg.build
** (RuntimeError) Error: app or version to deploy is not set.
Usually these are automatically picked up from Mix.Project.
If this is an umbrella app, you must set these in your deploy.exs, e.g.:
# config(:app, :myapp)
# config(:version, "0.0.1")
    deps/bootleg/lib/bootleg/tasks/build.exs:6: Bootleg.DynamicTasks.VerifyConfig.execute/0
    ...
```

sample projects
---------------

1. <https://github.com/smt116/elixir-umbrella-sample-application>
2. <https://github.com/shanesveller/kube-native-phoenix>
3. <https://github.com/joaquimadraz/opensubs.io>
4. <https://github.com/prasadtalasila/TransportScheduler/wiki/Umbrella-Application-Structure>

troubleshooting
---------------

### file navigation doesn't work in vim

that is you cannot "goto file" with `gf` command and cannot jump to alternate
file when using `vim-projectionist` plugin (or the like).

**solution**

change CWD to directory of child application - for example, use `cd` mapping
from inside NERDTree window.
