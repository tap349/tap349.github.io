---
layout: post
title: Jekyll - Tips
date: 2017-10-22 02:44:03 +0300
access: public
categories: []
---

<!-- more -->

* TOC
{:toc}
<hr>

## comments with Disqus

1. <http://sgeos.github.io/jekyll/disqus/2016/02/14/adding-disqus-to-a-jekyll-blog.html>
2. <https://tap349-blog.disqus.com/admin/settings/jekyll/>

- [register](https://disqus.com/admin/create/) site in Disqus
- [follow](https://disqus.com/admin/settings/jekyll/) instructions for Jekyll

adapted instructions:

- `comments: true` option

  `page.comments` variable is used to toggle comments for post.

  I generate new posts with Rake task, all of them having predefined
  YAML front matter. that is why it makes no sense to add this option
  to YAML front matter of post layout (it just won't be used).

  instead it was added (1) to the template in Rake task
  and (2) to each already existing post manually.

- [Universal Embed Code](https://disqus.com/admin/install/platforms/universalcode)

  save it, say, in _\_includes/disqus.html_:

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

      (function() {
        var d = document, s = d.createElement('script');
        // <website-name> is Website Name of your site in Disqus Settings
        s.src = 'https://<website-name>.disqus.com/embed.js';
        s.setAttribute('data-timestamp', +new Date());
        (d.head || d.body).appendChild(s);
      })();
    </script>

    <noscript>
      Please enable JavaScript to view the
      <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a>
    </noscript>
    <!-- end of pasted Universal Embed Code -->
  {% endif %}
  ```
  {% endraw %}

  and include at the end of _\_layouts/post.html_:

  ```html
  ...
  {% include disqus.html %}
  ```
