---
layout: post
title: Taskwarrior
date: 2018-11-10 14:41:25 +0300
access: private
comments: true
categories: [gtd]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

notes
-----

### description field width

description field width is about 90% of terminal width: say, terminal width
is 158 characters, then description field width is 140 characters.

taskwarrior calculates description field width using this formula (roughly):

```
max(MAX_LINE_LENGTH, 0.9 * TERMINAL_WIDTH)
```

here `MAX_LINE_LENGTH` is calculated by splitting the whole description into
words and finding the longest word. the point here is that only whitespaces
are used as delimiters - not newlines. in this case if you have several long
URLs each on its own line they are treated as a single very long word.

and if `MAX_LINE_LENGTH` turns out to be greater than `0.9 * TERMINAL_WIDTH`
report formatting is getting broken.

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
