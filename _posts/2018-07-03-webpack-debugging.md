---
layout: post
title: Webpack - Debugging
date: 2018-07-03 12:39:13 +0300
access: public
comments: true
categories: [webpack]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

debugging node modules
----------------------

it's safe to insert `debugger` anywhere in npm package source code - just make
sure that you insert `debugger` statement in the file that is getting compiled
by Webpack.

for example, npm package might contain multiple versions of the same JS file
(normal, minified, dist versions) but only one of them is specified in `main`
field of _package.json_ => this version is the one that is imported in your
application and compiled by Webpack eventually.
