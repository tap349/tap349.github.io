---
layout: post
title: React Native - Upgrading
date: 2017-11-20 11:33:05 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

post-upgrade instructions
-------------------------

```sh
$ rm -rf node_modules/
$ yarn cache clean
$ yarn
$ react-native run-ios
```

NOTE: make sure to rebuild applications for both iOS and Android.
