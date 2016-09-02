---
layout: post
title: download file from public in Rails
date: 2016-04-07 18:43:49 +0300
access: public
categories: [rails]
---

that is prevent browser from opening it instead.

view:

```slim
= link_to 'google.html',
  download_google_verification_file_site_diy_url(site),
  'data-no-turbolink': true,
  class: 'file'
```

controller:

```ruby
def download_google_verification_file
  file_path = Rails.root.join('public', 'google.html')
  send_file file_path, disposition: 'attachment'
end
```
