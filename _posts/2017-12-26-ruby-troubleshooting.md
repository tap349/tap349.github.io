---
layout: post
title: Ruby - Troubleshooting
date: 2017-12-26 19:23:05 +0300
access: public
comments: true
categories: [ruby]
---

<!-- more -->

* TOC
{:toc}
<hr>

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
------------------------------------------------------------------

the error occurred after installing new Ruby version (2.5.0):

```
$ bundle
...
An error occurred while installing json (1.8.3), and Bundler cannot continue.
Make sure that `gem install json -v '1.8.3'` succeeds before bundling.
```

**solution**

try to update offending gem:

```sh
$ bundle update json
```
