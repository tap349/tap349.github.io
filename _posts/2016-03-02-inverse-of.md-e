---
layout: post
title: inverse_of
date: 2016-03-02 11:08:00 +0300
access: public
comments: true
categories: [rails]
---

one example when `inverse_of` is useful.

spec:

```ruby
let(:site) { create :site }
let!(:gl_account) { create :gl_account, site: site }
```

in this case `gl_account` won't be found for `site` in ruby file:

```ruby
site.gl_account # nil
```

you would have to reload `site` to get access to `gl_account`:

```ruby
site.reload.gl_account # gl_account
```

using `inverse_of` on `site` association in `Google::Account` would help:

```ruby
belongs_to :site, inverse_of: :gl_account
```
