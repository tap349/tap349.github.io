---
layout: post
title: AppSignal - Notes
date: 2019-04-08 15:36:18 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

log location
------------

1. <https://docs.appsignal.com/elixir/configuration/options.html#option-working_directory_path>

> <https://docs.appsignal.com/support/debugging.html#log-location>
>
> The Elixir package will write the log files to the /tmp/appsignal.log
> log-file.

that is AppSignal log is not created in `working_directory_path` as I expected.
