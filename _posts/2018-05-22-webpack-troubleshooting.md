---
layout: post
title: Webpack - Troubleshooting
date: 2018-05-22 14:18:39 +0300
access: public
comments: true
categories: [webpack]
---

<!-- more -->

* TOC
{:toc}
<hr>

[Webpacker] Rails assets manifest file not found
------------------------------------------------

```
$ cap production deploy
...
03:04 deploy:assets:backup_manifest
      01 mkdir -p /home/sith/production/releases/20180522104043/assets_manifest_backup
      ...
      WARN  Rails assets manifest file not found.
...
Caused by:
Capistrano::FileNotFound: Rails assets manifest file not found.
```

**solution**

Capistrano runs `deploy:assets:precompile` and `deploy:assets:backup_manifest`
tasks when performing deployment if you require _capistrano/rails/assets_ file:

```ruby
# Capfile

require 'capistrano/rails/assets'
```

`deploy:assets:backup_manifest` task fails because it cannot find manifest in
_public/packs/_ - this is because Capistrano searches for manifest this way:

```ruby
# https://github.com/capistrano/rails/blob/master/lib/capistrano/tasks/assets.rake#L104

def detect_manifest_path
  %w(
    .sprockets-manifest*
    manifest*.*
  ).each do |pattern|
    candidate = release_path.join('public', fetch(:assets_prefix), pattern)
    return capture(:ls, candidate).strip.gsub(/(\r|\n)/,' ') if test(:ls, candidate)
  end
  msg = 'Rails assets manifest file not found.'
  warn msg
  fail Capistrano::FileNotFound, msg
end
```

by default `assets_prefix` option in Capistrano config (_config/deploy.rb_) is
set to `assets` so `deploy:assets:backup_manifest` task searches for manifest
inside _public/assets/_ directory which doesn't even exist: `webpacker:compile`
task creates _public/packs/_ directory (`public_output_path` option in Webpacker
config) to store compiled assets.

so there are 2 possible solutions to this problem:

- disable `deploy:assets:backup_manifest` task altogether

  1. <https://stackoverflow.com/a/48627238>
  2. <http://capistranorb.com/documentation/advanced-features/overriding-capistrano-tasks/>

  ```ruby
  # config/deploy.rb

  Rake::Task['deploy:assets:backup_manifest'].clear_actions
  ```

- *[RECOMMENDED]* set correct assets prefix

  1. <https://github.com/capistrano/rails#usage>

  ```ruby
  # config/deploy.rb

  set :assets_prefix, 'packs'
  ```
