---
layout: post
title: ImageMagick - Tips
date: 2017-10-18 12:11:39 +0300
access: public
categories: [imagemagick]
---

<!-- more -->

* TOC
{:toc}
<hr>

- get picture info

  ```sh
  $ identify wallet.png
  wallet.png PNG 32x32 32x32+0+0 8-bit sRGB 624B 0.000u 0:00.000
  ```

- convert SVG to PNG

  ```sh
  rsvg-convert -w 32 -h 32 wallet.svg > wallet.png
  ```