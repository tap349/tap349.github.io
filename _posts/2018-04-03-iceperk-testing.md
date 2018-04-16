---
layout: post
title: Iceperk - Testing
date: 2018-04-03 11:43:51 +0300
access: private
comments: true
categories: []
---

<!-- more -->

* TOC
{:toc}
<hr>

- phone numbers: +7 900 000-00-00 (and so forth)
- profile names: beta0001
- team names: beta0001 team

say, new release branch name is `release_3_16`.

### new server side + old app version

`iceperkapp`:

- change environment to `development` in _Env.js_
- switch to `develop` branch
- don't merge `release_3_16` branch into `develop` branch so far!

`iceperk`:

- merge `release_3_16` branch into `develop` branch
- merge `develop` branch into `master` branch
- switch to `master` branch

now it's possible to run all common tests to make sure nothing is broken.

### new server side + new app version

