---
layout: post
title: Yarn/npm - Troubleshooting
date: 2018-05-03 21:25:56 +0300
access: public
comments: true
categories: [yarn, npm, js]
---

<!-- more -->

* TOC
{:toc}
<hr>

The engine "node" is incompatible with this module
--------------------------------------------------

```
$ yarn
...[2/4] 🚚  Fetching packages...
error upath@1.0.4: The engine "node" is incompatible with this module. Expected version ">=4 <=9".
error Found incompatible module
```

**solution**

1. <https://github.com/gilbarbara/react-joyride/issues/131#issuecomment-253319816>

```sh
$ yarn --ignore-engines
```
