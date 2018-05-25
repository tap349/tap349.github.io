---
layout: post
title: Jekyll - Troubleshooting
date: 2017-07-21 01:54:32 +0300
access: public
comments: true
categories: [jekyll]
---

<!-- more -->

1. <http://downtothewire.io/2015/08/15/configuring-jekyll-for-user-and-project-github-pages/>

all styles are gone
-------------------

in Chrome Console:

```
GET http://tap349.github.io/postgresql/rails/2017/07/20/postgresql-tips/public/css/hyde.css
```

_\_includes/head.html_:

{% raw %}
```html
<link rel="stylesheet" href="{{ site.baseurl }}public/css/hyde.css">
```
{% endraw %}

it looks like `site.baseurl` is always set to current url - that's why
assets are searched for in wrong location.

**solution**

prefix all occurrences of `site.baseurl` with `site.url`
(effectively, use absolute urls instead of relative ones).

_\_includes/head.html_:

{% raw %}
```html
<link rel="stylesheet" href="{{ site.url }}/{{ site.baseurl }}public/css/hyde.css">
```
{% endraw %}

Page build failed
-----------------

in email from GitHub:

```
The page build failed for the `master` branch with the following error:
Page build failed. For more information, see ...
```

**solution**

build Jekyll site manually to find out the reason:

```sh
$ blog
$ jekyll build
```
