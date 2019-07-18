---
layout: post
title: AppSignal - Metrics
date: 2019-02-08 14:51:05 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://docs.appsignal.com/metrics/custom.html>

### collection

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

metric values are collected during this time frame (~1 minute) and then
final value (or values) is sent to AppSignal - it depends on metric type
what value is eventually persisted if helper is called multiple times:

- gauge: the latest value
- measurement: call count and average value
- counter: sum of counter values

for measurement metric both values are persisted - it creates 2 metric
fields (`COUNT` and `MEAN`) for them accordingly (4 metric fields in all).

### display

metric type also determines what values are used for current display time
frame in web UI - the same rules apply as when collecting metrics but now
these rules apply to persisted values (not the values passed to helpers).

### examples

- gauge: database size
- measurement: RPM and average response time
- counter: login count
