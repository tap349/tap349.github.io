---
layout: post
title: Markdown - Tips
date: 2017-05-28 01:15:46 +0300
access: public
categories: [markdown]
---

<!-- more -->

## backticks inside inline code block

<https://meta.stackexchange.com/questions/82718>

surround inline code block with double backticks and extra spaces:

    `` `IO.puts("Hello world")` ``

## escape Liquid template tags in Jekyll

- <https://stackoverflow.com/questions/24102498>
- <http://sarathlal.com/escape-liquid-tag-in-jekyll-posts>
- <https://stackoverflow.com/questions/3426182>

wrap code block with `{{ "{% raw " }}%}` and `{{ "{% endraw " }}%}` tags:

    {{ "{% raw " }}%}{% raw %}
    ```html
    <link rel="stylesheet" href="{{ site.baseurl }}public/css/hyde.css">
    ```
    {% endraw %}{{ "{% endraw " }}%}
