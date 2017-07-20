---
layout: post
title: Jekyll troubleshooting
date: 2017-07-21 01:54:32 +0300
access: public
categories: [jekyll]
---

<!-- more -->

- <http://downtothewire.io/2015/08/15/configuring-jekyll-for-user-and-project-github-pages/>

## all styles are lost

in Chrome Console:

```
GET http://tap349.github.io/postgresql/rails/2017/07/20/postgresql-tips/public/css/hyde.css
```

_\_includes/head.html_:

```html
<link rel="stylesheet" href="{{ site.baseurl }}public/css/hyde.css">
```

it looks like `site.baseurl` is always set to current url - that is why
assets are searched for in wrong location.

**solution**

prefix all occurrences of `site.baseurl` with `site.url`
(no idea why it works - just a quick fix).

_\_includes/head.html_:

```html
<link rel="stylesheet" href="{{ site.url }}/{{ site.baseurl }}public/css/hyde.css">
```
