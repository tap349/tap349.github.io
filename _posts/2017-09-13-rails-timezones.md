---
layout: post
title: Rails - Timezones
date: 2017-09-13 14:03:31 +0300
access: public
categories: [rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

- parse datetime string wo timezone in specified timezone

  ```ruby
  > ActiveSupport::TimeZone['Europe/Moscow'].parse('2017-09-13 13:50:00')
  Wed, 13 Sep 2017 13:50:00 MSK +03:00
  > _.to_s(:db)
  "2017-09-13 10:50:00"
  ```
