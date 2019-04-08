---
layout: post
title: AppSignal - Troubleshooting
date: 2019-04-08 15:26:25 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

### deploy is not visible

when revision is bumped in _config/appsignal.exs_ and application is deployed,
deploy marker is not created - that is new deploy is not shown in the list of
deploys and new incoming data is not tagged to that deploy accordingly.

**solution**

deploy marker is created the first time error is reported to AppSignal only.
