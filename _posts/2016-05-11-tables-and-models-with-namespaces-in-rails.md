---
layout: post
title: tables and models with namespaces in Rails
date: 2016-05-11 19:03:23 +0300
access: public
categories: [rails]
---

in Rails modules are namespaces - I will use the term `module` onward only.

when searching for table ActiveRecord always uses model name without modules.
e.g. for `Stats::Google::Analytics::McfStat` it will search for `mcf_stats` table.

to use module name as table name prefix it's necessary to define
table name prefix for this module:

- create file named as top-level module
- define table name prefix for top-level module or any nested module

e.g. for `Stats::Google::Analytics::McfStat` create _app/models/stats/stats.rb_:

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

as shown above table name prefix doesn't have to match module hierarchy.

**NOTE**: don't use `::` notation when defining table name prefixes!

### different table name prefixes for different nested modules

table name prefix for high level module will override one for any nested module!

to have different table name prefixes for different nested modules define all
of them in the same file starting with table name prefix for inner-most module -
this way it's not overriden with table name prefixes for modules at higher levels.

e.g. in _app/models/stats/stats.rb_: 

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

module Stats
  def self.table_name_prefix
    'stats_'
  end
end
```
