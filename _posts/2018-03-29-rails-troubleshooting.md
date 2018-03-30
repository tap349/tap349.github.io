---
layout: post
title: Rails - Troubleshooting
date: 2018-03-29 17:02:54 +0300
access: public
comments: true
categories: [rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

null array elements are removed in params
-----------------------------------------

1. <https://gist.github.com/subfuzion/08c5d85437d5d4f00e58#post>

data is sent in JSON format via POST request.

`null` array elements are present when reading request body directly:

```ruby
# in controller
request.body.read
```

but they are already removed in parsed params:

```ruby
# in controller
params.to_unsafe_h
```

**solution**

1. <https://stackoverflow.com/a/25428800/3632318>
2. <https://github.com/rails/rails/commit/e8572cf2f94872d81e7145da31d55c6e1b074247>

probably this has something to do with `perform_deep_munge` Rails feature -
you might try to disable it:

```ruby
config.action_dispatch.perform_deep_munge = false
```

but I've chosen to send `null` array elements as empty strings - they are
not removed by Rails and are properly converted to `nil`s by `Form` schema
of dry-validation.