---
layout: post
title: cancancan
date: 2016-04-03 15:42:14 +0300
access: public
categories: [rails, cancancan]
---

<https://github.com/ryanb/cancan/wiki/authorizing-controller-actions>

when using `load_and_authorize_resource` cancancan tries to guess resource
class based on controller name.

but if they differ it will fail to do so and will prevent controller
from rendering a view - silently!

_app/models/billing/accounting_act.rb_:

```ruby
class AccountingAct
  ...
end
```

_app/controllers/billing/accounting_acts_controller.rb_:

```ruby
class Billing::AccountingActsController
  load_and_authorize_resource
  ...
end
```

in this case CanCanCan will try to load `Billing::AccountingAct` resource
but will not raise any errors when it doesn't find appropriate model.

to fix it specify resource name explicitly:

_app/controllers/billing/accounting_acts_controller.rb_:

```ruby
class Billing::AccountingActsController
  load_resource :accounting_act
  authorize_resource
  ...
end
```
