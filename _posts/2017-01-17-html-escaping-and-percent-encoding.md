---
layout: post
title: HTML escaping and percent-encoding
date: 2017-01-17 14:23:20 +0300
access: public
comments: true
categories: [phoenix, rails, html]
---

<!-- more -->

* TOC
{:toc}
<hr>

HTML escaping
-------------

used to escape `&`, `<`, `>` and `"` in HTML document.

### encode (escape, html-escape)

#### Rails

- `CGI.escapeHTML`

  ```ruby
  CGI.escapeHTML 'hello "world"!'
  # => "hello &quot;world&quot;!"
  ```

- `raw` helper in views

  ```ruby
  = raw 'hello "world"!'
  ```

- `Rails::Html::FullSanitizer.new.sanitize`

  though its primary purpose is to sanitize HTML string (inter alia,
  to remove all HTML tags) it can also be used to unescape HTML entities
  which is part of sanitization process (with some caveats - see below).

  ```ruby
  Rails::Html::FullSanitizer.new.sanitize("hello &gt; &quot;world&quot;")
  # => "hello &gt; \"world\""
  ```

  NOTE: it doesn't unescape all HTML entities (say, `&lt;`/`&gt;`)!

#### Phoenix

1. <https://github.com/martinsvalin/html_entities>

- `Phoenix.HTML.html_escape/1`

  ```elixir
  ~s(hello "world")
  |> Phoenix.HTML.html_escape()
  |> Phoenix.HTML.safe_to_string()
  # => "hello &quot;world&quot;"
  ```

- `HtmlEntities.encode/1`

  ```elixir
  ~s(hello "world")
  |> HtmlEntities.encode()
  # => "hello &quot;world&quot;"
  ```

### decode (unescape)

#### Rails

- `CGI.unescapeHTML`

  ```ruby
  CGI.unescapeHTML('hello &quot;world&quot;!')
  # => "hello \"world\"!"
  ```

#### Phoenix

- `HtmlEntities.decode/1`

  ```elixir
  "hello &quot;world&quot;"
  |> HtmlEntities.decode()
  # => "hello \"world\""
  ```

percent-encoding (percent-escaping, URL encoding)
-------------------------------------------------

1. <https://en.wikipedia.org/wiki/Percent-encoding>

used to encode both URLs (primarily query strings) and HTML form data
of `application/x-www-form-urlencoded` media type (in the latter case
reserved characters are always encoded).

- reserved characters

  ```
  [!*'();:@&=+$,/?#[\]]
  ```

  they are percent-encoded only when they have no special meaning
  (that is they are always percent-encoded in query param values).

- unreserved characters

  ```
  [a-zA-Z0-9\-_.~]
  ```

  they are never percent-encoded (they have the same meaning when
  both escaped and unescaped).

- other characters

  they are always percent-encoded.

test URL: `http://test.com?q1=foo,bar&q2=http://foo.com`.

### encode (percent-encode, escape, percent-escape)

#### Rails

- `Addressable::URI.escape`

  it doesn't escape reserved characters such as `:` and `/`:

  ```ruby
  Addressable::URI.escape('http://test.com?q1=foo,bar&q2=http://foo.com')
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"
  ```

  it's equivalent to `URI.encode/2` in Phoenix.

- `CGI.escape`

  it escapes all characters that can be escaped (that is includes reserved
  characters unlike `Addressable::URI.escape`):

  ```ruby
  CGI.escape('http://test.com?q1=foo,bar&q2=http://foo.com')
  # => "http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

- `ERB::Util.url_encode`

  it does the same as `CGI.escape`:

  ```ruby
  ERB::Util.url_encode('http://test.com?q1=foo,bar&q2=http://foo.com')
  # => "http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

- `#to_query` (aliased to `#to_param`)

  it converts hash into a query string and escapes all characters
  like `CGI.escape` or `ERB::Util.url_encode`:

  ```ruby
  { redirect_uri: 'http://test.com?q1=foo,bar&q2=http://foo.com' }.to_query
  # => "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

- `URI.encode_www_form`

  it does the same as `#to_query` but accepts array of arrays instead of hash:

  ```ruby
  URI.encode_www_form([['redirect_uri', 'http://test.com?q1=foo,bar&q2=http://foo.com']])
  # => "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

- `Addressable::URI.form_encode`

  it's like a combination of `#to_query` and `URI.encode_www_form` (in that
  it accepts both hash and array of arrays):

  ```ruby
  Addressable::URI.form_encode({ 'redirect_uri' => 'http://test.com?q1=foo,bar&q2=http://foo.com' })
  # => "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"

  Addressable::URI.form_encode([['redirect_uri', 'http://test.com?q1=foo,bar&q2=http://foo.com']])
  # => "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

#### Phoenix

- `URI.encode/2`

  <https://hexdocs.pm/elixir/URI.html#encode/2>:

  > Percent-escapes all characters that require escaping in a string.
  >
  > reserved characters, such as : and /, and the so-called unreserved
  > characters, which have the same meaning both escaped and unescaped,
  > won’t be escaped by default.

  ```elixir
  "http://test.com?q1=foo,bar&q2=http://foo.com"
  |> URI.encode()
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"
  ```

  `URI.encode/2` would escape whitespace (as `%20`) and all characters
  except for reserved and unreserved ones - that's all it does as far
  as I can judge.

  => it's equivalent to `Addressable::URI.escape` in Rails.

- `URI.encode_www_form/1`

  ```elixir
  "http://test.com?q1=foo,bar&q2=http://foo.com"
  |> URI.encode_www_form()
  # => "http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

  => it's equivalent to `CGI.escape` or `ERB::Util.url_encode` in Rails.

- `URI.encode_query/1`

  ```elixir
  %{"redirect_uri" => "http://test.com?q1=foo,bar&q2=http://foo.com"}
  |> URI.encode_query()
  # => "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  ```

  => it's equivalent to `#to_query` or `Addressable::URI.form_encode` in Rails.

### decode (percent-decode, unescape, percent-unscape)

all decode functions in both Rails and Phoenix decode ALL escaped characters -
they differ only in how they return the result (string, hash or array of arrays).

#### Rails

- `CGI.unescape`

  ```ruby
  CGI.unescape('http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com')
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"
  ```

- `URI.decode_www_form`

  ```ruby
  URI.decode_www_form('redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com')
  # => [
  #      [
  #        "redirect_uri",
  #        "http://test.com?q1=foo,bar&q2=http://foo.com"
  #      ]
  #    ]
  ```

- `Addressable::URI.form_unencode`

  ```ruby
  Addressable::URI.form_unencode('redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com')
  # => [
  #      [
  #        "redirect_uri",
  #        "http://test.com?q1=foo,bar&q2=http://foo.com"
  #      ]
  #    ]
  ```

#### Phoenix

- `URI.decode/1`

  ```elixir
  "http://test.com?q1=foo,bar&q2=http://foo.com"
  |> URI.decode()
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"

  "http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  |> URI.decode()
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"
  ```

- `URI.decode_www_form/1`

  ```elixir
  "http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  |> URI.decode_www_form()
  # => "http://test.com?q1=foo,bar&q2=http://foo.com"
  ```

- `URI.decode_query/2`

  ```elixir
  "redirect_uri=http%3A%2F%2Ftest.com%3Fq1%3Dfoo%2Cbar%26q2%3Dhttp%3A%2F%2Ffoo.com"
  |> URI.decode_query()
  # => %{"redirect_uri" => "http://test.com?q1=foo,bar&q2=http://foo.com"}
  ```
