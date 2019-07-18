---
layout: post
title: dry-rb - dry-struct
date: 2018-06-15 13:53:24 +0300
access: public
comments: true
categories: [dry-rb]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

use default value for nil
-------------------------

1. <http://dry-rb.org/gems/dry-struct/recipes/#resolving-default-values-on-nil>

NOTE: dry-struct v0.5.0 is required.

```ruby
class Fb::AdStatStruct < Dry::Struct::Value
  transform_types do |type|
    # if type has default value
    if type.default?
      type.constructor do |value|
        # Dry::Types::Undefined triggers default value
        value.nil? ? Dry::Types::Undefined : value
      end
    else
      type
    end
  end

  attribute :comments, Types::Coercible::Integer.default(0)
  attribute :likes, Types::Coercible::Integer.default(0)
  attribute :post_reactions, Types::Coercible::Integer.default(0)
end
```
