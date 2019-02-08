---
layout: post
title: AppSignal - Metrics
date: 2019-02-08 14:51:05 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://docs.appsignal.com/metrics/custom.html>

> <https://docs.appsignal.com/appsignal/request-lifecycle.html>
>
> After your application serves the request, the transaction is closed and
> placed into a queue. Depending on your environment (development/production)
> and the gem version, the thread that was started during the application
> start sleeps a certain amount of time. Once the sleep is over it processes
> the queue of transactions and sends it to our AppSignal Push API.

custom metrics are not sent immediately - they must be buffered by agent
during some amount of time (most likely 1 minute) and only then are they
sent to AppSignal.

=> all metric values are collected during this time frame (1 minute) and
then final value is sent to AppSignal - it depends on metric type how this
final value is obtained:

- gauge: the latest value
- measurement: sum of calls and mean value (2 values)
- counter: sum of values

measurement metric creates 2 metrics with `_count` and `_mean` suffixes -
that is why 2 values are sent.

examples:

- gauge: database size
- measurement: RPM and average response time
- counter: login count
