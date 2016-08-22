---
layout: post
title: RSpec troubleshooting
date: 2016-08-22 14:17:21 +0300
access: public
categories: [rspec, troubleshooting]
---

RSpec related issues.

<!-- more -->

### `NameError: uninitialized constant TestService`

#### description

RSpec cannot find any project classes.

#### background

by default `rails generate rspec:install` creates this _.rspec_:

```ruby
--color
--require spec_helper
```

but all project files are required in _spec/rails_helper.rb_ using this line:

```ruby
require File.expand_path('../../config/environment', __FILE__)
```

also _spec/rails_helper.rb_ requires _spec/spec_helper.rb_.

#### solution

just require `rails_helper` instead of `spec_helper` in _.rspec_:

```ruby
--color
--require rails_helper
```
