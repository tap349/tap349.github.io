---
layout: post
title: tables and models within modules in Rails
date: 2016-05-11 19:03:23 +0300
access: public
comments: true
categories: [rails]
---

about models within modules and corresponding table names in Rails.

<!-- more -->

when searching for table AR always uses model name without modules.
e.g. for `Stats::Google::Analytics::McfStat` it will search for `mcf_stats` table.

**NOTE**: AR still finds table `sites_descriptions` for `Site::Description` model -
          when module (`Site`) is a class of existing model (`Site::Description`).

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

table name prefix for high level module will override one for
any nested module if defined *before* the latter:

_app/models/stats/stats.rb_:

```ruby
module Stats
  def self.table_name_prefix
    'stats_'
  end
end

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

to have different table name prefixes for different nested modules:

- either define table name prefixes starting with innermost module:

  _app/models/stats/stats.rb_:

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
- or define table name prefixes for all modules inside the same definition:

  _app/models/stats/stats.rb_:

  ```ruby
  module Stats
    def self.table_name_prefix
      'stats_'
    end

    module Google
      module Analytics
        def self.table_name_prefix
          'google_analytics_'
        end
      end
    end
  end
  ```
