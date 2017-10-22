---
layout: post
title: URL-encode/decode strings
date: 2016-07-15 19:15:52 +0300
access: public
comments: true
categories: [rails, html]
---

ways to:

- generate URL-encoded string (used primarily for query strigs) from hash,
  array or string
- decode it back into either array or string

NOTE: this type of encoding is sometimes referred to as percent-encoding.

<!-- more -->

NOTE: all functions expect query string (its form may vary depending on specific
      function) to be passed as parameter - if passed complete URL they will either
      try to encode the whole URL (for `encode`) or do nothing (for `decode`).

## encode

#### hash

- `#to_query` (aliased to `#to_param`):

  ```ruby
  (dev)> { redirect_url: 'http://test.com?message=привет' }.to_query
  "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

- `Addressable::URI.form_encode`:

  ```ruby
  (dev)> Addressable::URI.form_encode({ 'redirect_url' => 'http://test.com?message=привет' })
  "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

#### array

- `URI.encode_www_form`:

  ```ruby
  (dev)> URI.encode_www_form [['redirect_url', 'http://test.com?message=привет']]
  "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

- `Addressable::URI.form_encode`:

  ```ruby
  (dev)> Addressable::URI.form_encode [['redirect_url', 'http://test.com?message=привет']]
  "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

#### string

- `CGI.escape`:

  ```ruby
  (dev)> CGI.escape 'redirect_url=http://test.com?message=привет'
  "redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

- `ERB::Util.url_encode`:

  ```ruby
  (dev)> ERB::Util.url_encode 'redirect_url=http://test.com?message=привет'
  "redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

## decode

#### to array

- `URI.decode_www_form`:

  ```ruby
  (dev)> URI.decode_www_form "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  [
      [0] [
          [0] "redirect_url",
          [1] "http://test.com?message=привет"
      ]
  ]
  ```

- `Addressable::URI.form_unencode`:

  ```ruby
  (dev)> Addressable::URI.form_unencode "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  [
      [0] [
          [0] "redirect_url",
          [1] "http://test.com?message=привет"
      ]
  ]
  ```

#### to string

`CGI.unescape`:

```ruby
(dev)> CGI.unescape 'redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82'
"redirect_url=http://test.com?message=привет"
```
