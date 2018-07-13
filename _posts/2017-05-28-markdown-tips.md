---
layout: post
title: Markdown - Tips
date: 2017-05-28 01:15:46 +0300
access: public
comments: true
categories: [markdown]
---

<!-- more -->

* TOC
{:toc}
<hr>

backticks inside inline code block
----------------------------------

<https://meta.stackexchange.com/questions/82718>

surround inline code block with double backticks and extra spaces:

    `` `IO.puts("Hello world")` ``

(how to) escape Liquid template tags in Jekyll
----------------------------------------------

1. <https://stackoverflow.com/questions/24102498>
2. <http://sarathlal.com/escape-liquid-tag-in-jekyll-posts>
3. <https://stackoverflow.com/questions/3426182>

wrap code block with `{{ "{% raw " }}%}` and `{{ "{% endraw " }}%}` tags:

    {{ "{% raw " }}%}{% raw %}
    ```html
    <link rel="stylesheet" href="{{ site.baseurl }}public/css/hyde.css">
    ```
    {% endraw %}{{ "{% endraw " }}%}

also it's possible to wrap arbitrary text with `{{ "{% raw " }}%}` and
`{{ "{% endraw " }}%}` tags:

    {{ "{% raw " }}%}{% raw %}
    both `{% if ... %}` and `{% endif %}` statements indent the next line
    {% endraw %}{{ "{% endraw " }}%}

escape left angle bracket only
------------------------------

it's not required to escape right angle bracket:

```
=>
\<foo>
```

don't escape anything inside blocks of code.
