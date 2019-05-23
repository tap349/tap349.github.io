---
layout: post
title: Browser Bugs
date: 2018-10-09 23:42:14 +0300
access: private
comments: true
categories: [opera, yandex, vivaldi]
---

<!-- more -->

* TOC
{:toc}
<hr>

Opera (56.0.3051.36)
--------------------

- application doesn't respond to any shortcuts after closing `Find in page`
  popup - it's necessary to focus application window manually with a mouse

  ***UPDATE (2019-05-23)***

  bug is fixed in Opera 60.0.3255.109.

Yandex (18.9.0)
---------------

- huge memory footprint compared to Opera or Vivaldi
- `Find on current page` search works with a delay - not instantaneously
  like in most other browsers (Google Chrome or Opera)
- background color is changed to gray when address bar is focused - I know
  it's by design but still it seems to me very distracting

Vivaldi (2.0.1309.37)
---------------------

after customizing interface in _custom.css_ everything seemed to be great
but still:

- when you start typing in address bar, you are presented with a list of
  suggestions but it's impossible to navigate between them with familiar
  `<C-p>`/`<C-n>` Emacs shortcuts - only with arrows (just like in FF)
