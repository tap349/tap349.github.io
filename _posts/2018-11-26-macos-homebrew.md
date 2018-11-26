---
layout: post
title: macOS - Homebrew
date: 2018-11-26 18:20:34 +0300
access: public
comments: true
categories: [macos]
---

<!-- more -->

* TOC
{:toc}
<hr>

### downgrade to previous formula version

1. <https://jaketrent.com/post/downgrade-previously-installed-brew-formula>

```sh
$ brew list --versions macvim
macvim 8.1-151 8.1-149_1 8.1-150 8.1-151_1 8.0-146_1 8.1-149 8.1-147 8.1-148
$ brew switch macvim 8.1-151
```
