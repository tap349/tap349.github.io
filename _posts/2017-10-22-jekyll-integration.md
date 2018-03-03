---
layout: post
title: Jekyll - Integration
date: 2017-10-22 02:44:03 +0300
access: public
comments: true
categories: [jekyll]
---

<!-- more -->

* TOC
{:toc}
<hr>

Disqus
------

1. <http://sgeos.github.io/jekyll/disqus/2016/02/14/adding-disqus-to-a-jekyll-blog.html>
2. <https://tap349-blog.disqus.com/admin/settings/jekyll/>

- [register](https://disqus.com/admin/create/) site in Disqus
- [follow](https://disqus.com/admin/settings/jekyll/) instructions for Jekyll

my instructions:

- add Universal Embed Code to post layout

  1. <https://disqus.com/admin/install/platforms/universalcode>

  save it, say, in _\_includes/disqus.html_ and edit as follows:

  {% raw %}
  ```html
  <!--
  include comments only if they are enabled for post
  (`comments: true` in YAML Front Matter)
  -->
  {% if page.comments %}
    <!-- beginning of pasted Universal Embed Code -->
    <div id="disqus_thread"></div>

    <script>
      var disqus_config = function () {
        // <site-url> is your site url (see `url` option in _config.yml)
        this.page.url = "<site-url>{{ page.url }}";
        this.page.identifier = "{{ page.id }}";
      };

      ...
    </script>

    ...
    <!-- end of pasted Universal Embed Code -->
  {% endif %}
  ```
  {% endraw %}

  include that file at the very end of post layout (_\_layouts/post.html_):

  {% raw %}
  ```html
  ...

  {% include disqus.html %}
  ```
  {% endraw %}

- add `comments: true` option to each post

  `page.comments` variable is used to toggle comments for post
  (in _\_includes/disqus.html_).

  adding this option to YAML front matter of post layout
  (_\_layouts/post.html_) had no effect so I added it to
  post template in Rake task (used to generate new posts)
  and to each already existing post manually.

Google Analytics
----------------

1. <https://desiredpersona.com/google-analytics-jekyll/>

- create account and new property (= your website) in GA

  | `Property` → `Tracking Info` → `Tracking Code`

  in the end you'll be provided with Tracking ID and tracking code
  (Global Site Tag) that can be pasted into the `<HEAD>` of every
  webpage you want to track with GA.

- add tracking code to `<HEAD>` of default layout

  save tracking code, say, in _\_includes/ga.html_:

  {% raw %}
  ```html
  <!--
  default Jekyll environment is development -
  it's production when built on GitHub Pages
  -->
  {% if jekyll.environment == 'production' %}
    <!-- beginning of pasted Global Site Tag -->
    ...
    <!-- end of pasted Global Site Tag -->
  {% endif %}
  ```
  {% endraw %}

  include that file at the very end of `<HEAD>` section in
  _\_includes/head.html_ (which is included in default layout).

  {% raw %}
  ```html
  <head>
    ...

    {% include ga.html %}
  </head>
  ```
  {% endraw %}
