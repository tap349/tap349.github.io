---
layout: post
title: Elixir - Umbrella
date: 2019-04-04 21:51:44 +0300
access: public
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

notes
-----

> <https://news.ycombinator.com/item?id=17117155>
>
> Basically you could split up your apps and hack on them independently (but
> actually have a good way to share dependencies if needed) and at deploy time
> you could choose to deploy all of them, or just the ones you want.

tips
----

### don't configure umbrella application

you cannot add any options to `umbrella_app` configuration:

```elixir
# umbrella_app/config/config.exs

config :umbrella_app, foo: 123
```

you'll this warning during compilation as a result:

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
its project configuration as well (`project/0` function inside _mix.exs_).

deployment
----------

1. <https://hexdocs.pm/distillery/introduction/umbrella_projects.html>

### Distillery

> <https://hackernoon.com/mastering-elixir-releases-with-distillery-a-pretty-complete-guide-497546f298bc#4eae>
>
> Distillery is also great when working with Umbrella apps! There are really
> only two things you need to pay attention to:
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

sample projects
---------------

1. <https://github.com/smt116/elixir-umbrella-sample-application>
2. <https://github.com/shanesveller/kube-native-phoenix>
3. <https://github.com/joaquimadraz/opensubs.io>
