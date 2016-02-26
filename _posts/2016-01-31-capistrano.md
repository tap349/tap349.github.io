---
layout: post
title: capistrano
date: 2016-01-31 23:39:00 +0300
access: public
categories: [capistrano]
---

https://github.com/capistrano/capistrano/blob/master/README.md

- It's just SSH
  Everything in Capistrano comes down to running SSH commands on remote servers.

- Key-based SSH
  Capistrano depends on connecting to your server(s) with SSH using key-based (i.e. password-less) authentication.

- Not for provisioning
  Server provisioning steps are not done by Capistrano.

Problems when deploying:

- first run of `deploy:assets:precompile` task exited with error `ExecJS::RuntimeError: (execjs):1`

  on the next run assets were precompiled without errors.

- capistrano runs rake `db:migrate` on first deploy instead of rake `db:schema:load`

  one option is to comment out `require 'capistrano/rails/migrations'` line in _Capfile_,
  run `RAILS_ENV=staging rake db:schema:load` manually and uncomment the line afterwards.

- `sidekiq:start` task hangs, just hangs

  problem was solved by removing command to source rvm script from sidekiq init.d script.
