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
