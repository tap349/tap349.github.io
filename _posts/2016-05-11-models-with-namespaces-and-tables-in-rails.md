---
layout: post
title: models with namespaces and tables in Rails
date: 2016-05-11 19:03:23 +0300
access: public
categories: [rails]
---

when searching for table ActiveRecord always uses model name without namespaces.
e.g. for `Stats::Google::Analytics::McfStat` it will search for `mcf_stats` table.

to use modules as table prefixes it's necessary to define `table_name_prefix`:

- create file named as top-level module
- define table name prefix for any module inside top-level module

e.g. for `Stats::Google::Analytics::McfStat` create _app/models/stats.rb_:

```ruby
module Stats
  module Google
    module Analytics
      def self.table_name_prefix
        'google_analytics_'
      end
    end
  end
end
```

as shown above table name prefix doesn't have to match module hierarchy exactly.

**NOTE**: table name prefix for high level module will override one for any nested module!

**NOTE**: don't use `::` notation when defining custom table name prefixes!
