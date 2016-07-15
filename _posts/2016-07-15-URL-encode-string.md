---
layout: post
title: URL-encode/decode query string
date: 2016-07-15 19:15:52 +0300
access: public
categories: [rails]
---

ways to:

- generate URL-encoded query string from hash, array or string
- decode it back into either array or string

<!-- more -->

## encode

#### hash

`#to_query` (aliased to `#to_param`):

```ruby
pumba(dev)> { redirect_url: 'http://test.com?message=привет' }.to_query
"redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
```

#### array

`URI.encode_www_form`:

```ruby
pumba(dev)> URI.encode_www_form [['redirect_url', 'http://test.com?message=привет']]
"redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
```

#### string

- `CGI.escape`:

  ```ruby
  pumba(dev)> CGI.escape 'redirect_url=http://test.com?message=привет'
  "redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

- `ERB::Util.url_encode`:

  ```ruby
  pumba(dev)> ERB::Util.url_encode 'redirect_url=http://test.com?message=привет'
  "redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
  ```

## decode

#### to array

`URI.decode_www_form`:

```ruby
pumba(dev)> URI.decode_www_form "redirect_url=http%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82"
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
pumba(dev)> CGI.unescape 'redirect_url%3Dhttp%3A%2F%2Ftest.com%3Fmessage%3D%D0%BF%D1%80%D0%B8%D0%B2%D0%B5%D1%82'
"redirect_url=http://test.com?message=привет"
```
