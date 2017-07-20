---
layout: post
title: Markdown tips
date: 2017-05-28 01:15:46 +0300
access: public
categories: [markdown]
---

<!-- more -->

## backticks inside inline code block

<https://meta.stackexchange.com/questions/82718>

surround inline code block with double backticks and extra spaces:

<pre>
`` `IO.puts("Hello world")` ``
</pre>

## double curly braces inside code block (Jekyll only)

<https://stackoverflow.com/questions/24102498>

> Any liquid tags inside this block don't get executed and are displayed as is.

wrap code block with `{% raw %}` and `{% endraw %}` tags:

<pre>
{% raw %}
```html
\<link rel="stylesheet" href="{{ site.baseurl }}public/css/hyde.css"\>
```
{% endraw %}
</pre>
