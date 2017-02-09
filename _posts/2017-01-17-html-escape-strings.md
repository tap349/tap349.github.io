---
layout: post
title: HTML-escape/unescape strings
date: 2017-01-17 14:23:20 +0300
access: public
categories: [rails, html]
---

ways to:

- escape special characters in HTML string (namely `&"<>`)
- unescape HTML entities in HTML-escaped string (`&quot;` => `"`, `&lt;` => `<`, etc.)

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

- `Rails::Html::FullSanitizer.new.sanitize`

  though its primary purpose is to sanitize HTML string
  it can also be used to unescape HTML entities which is part
  of sanitization process (with some caveats - see below).

  ```ruby
  (dev)> Rails::Html::FullSanitizer.new.sanitize("hello &gt; &quot;world&quot;")
  "hello &gt; \"world\""
  ```

  NOTE: it doesn't unescape all HTML entities! (e.g. as is the case with `&lt;`/`&gt;`)

## unescape

- `CGI.unescapeHTML`

  ```ruby
  (dev)> CGI.unescapeHTML 'hello &quot;world&quot;!'
  "hello \"world\"!"
  ```
