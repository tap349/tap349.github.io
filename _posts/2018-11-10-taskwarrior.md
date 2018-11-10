---
layout: post
title: Taskwarrior
date: 2018-11-10 14:41:25 +0300
access: private
comments: true
categories: [gtd]
---

<!-- more -->

* TOC
{:toc}
<hr>

notes
-----

currently task description size is limited to 158 characters:

```
$ task
...
The report has a minimum width of 196 and does not fit in the available width of 158.
```

tips
----

### number of tasks to show

1. <https://www.deimeke.net/dirk/blog/uploads/taskwarrior/task.ref.pdf>

> man task
>
> limit:<number-of-rows>
>        Specifies the desired number of tasks a report should show, if a
>        positive integer is given. The value 'page' may also be used, and
>        will limit the report output to as many lines of text as will fit
>        on screen. This defaults to 25 lines.

it looks like `page` value is used by default. specify `limit` attribute to
show more tasks:

```sh
$ task limit:20
```
