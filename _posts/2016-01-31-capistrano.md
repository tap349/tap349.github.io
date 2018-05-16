---
layout: post
title: capistrano
date: 2016-01-31 23:39:00 +0300
access: public
comments: true
categories: [capistrano]
---

miscellaneous notes about capistrano (general information, troubleshooting).

<!-- more -->

notes
-----

<https://github.com/capistrano/capistrano/blob/master/README.md>

- It's just SSH

  Everything in Capistrano comes down to running SSH commands on remote servers.

- Key-based SSH

  Capistrano depends on connecting to your server(s)
  with SSH using key-based (i.e. password-less) authentication.

- Not for provisioning

  Server provisioning steps are not done by Capistrano.

troubleshooting
---------------

first run of `deploy:assets:precompile` task exited with error `ExecJS::RuntimeError: (execjs):1`
-------------------------------------------------------------------------------------------------

on the next run assets were precompiled without errors.

capistrano runs rake `db:migrate` on first deploy instead of rake `db:schema:load`
----------------------------------------------------------------------------------

one option is to comment out `require 'capistrano/rails/migrations'` line in
_Capfile_, run `RAILS_ENV=staging rake db:schema:load` manually and uncomment
the line afterwards.

`sidekiq:start` task hangs, just hangs
--------------------------------------

problem was solved by removing command to source RVM script from Sidekiq
init.d script.

fatal: Could not read from remote repository.
---------------------------------------------

```
$ cap production deploy
...
00:05 git:check
      01 git ls-remote git@github.com:Complead/sith.git HEAD
      01 Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
      01 git@github.com: Permission denied (publickey).
      01 fatal: Could not read from remote repository.
      01
      01 Please make sure you have the correct access rights
      01 and the repository exists.
```

**solution**

1. <https://stackoverflow.com/a/15940055/3632318>

first make sure you forward agent in _.ssh/config_:

```
Host sith
 User sith
 Hostname <hostname>
 IdentityFile ~/.ssh/id_rsa
 ForwardAgent yes
```

then it's necessary to restart SSH agent: either restart terminal if you're
using `ssh-agent` Zsh plugin or restart SSH agent manually:

```sh
$ ssh-add -D
$ ssh-agent
$ ssh-add
```
