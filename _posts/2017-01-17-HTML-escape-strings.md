---
layout: post
title: HTML-escape/unescape strings
date: 2017-01-17 14:23:20 +0300
access: public
categories: [rails, html]
---

ways to:

- escape special characters in HTML string (namely `&"<>`)
- unescape HTML-escaped string back (`&quot;` => `"`, `&lt;` => `<`, etc.)

<!-- more -->

## escape

- `CGI.escapeHTML`

  ```ruby
  (dev)> CGI.escapeHTML 'hello "world"!'
  "hello &quot;world&quot;!"
  ```

- `raw` (helper in views)

  ```ruby
  = raw 'hello "world"!'
  ```

## unescape

- `CGI.escapeHTML`

  ```ruby
  (dev)> CGI.unescapeHTML 'hello &quot;world&quot;!'
  "hello \"world\"!"
  ```
