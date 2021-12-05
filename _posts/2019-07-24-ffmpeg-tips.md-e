---
layout: post
title: FFmpeg - Tips
date: 2019-07-24 09:21:53 +0300
access: public
comments: true
categories: [ffmpeg]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## cut video

1. <https://stackoverflow.com/a/42827058/3632318>

```sh
$ ffmpeg -i input.mkv -ss 00:10:11 -to 00:12:31 -c copy S01E01_1.mkv
```

NOTE: `-t` option specifies duration, `-to` option specifies end time (which is
more convenient).
