---
layout: post
title: Chef - Quickref (using chef-zero and knife-zero)
date: 2016-08-31 14:32:47 +0300
access: public
categories: [chef, knife-zero]
---

refined and focused quickref for `chef-zero` and `knife-zero`.

<!-- more -->

- install ChefDK and `knife-zero` on workstation

  ```sh
  (ws)$ brew update
  (ws)$ brew cask reinstall chefdk
  (ws)$ chef gem install knife-zero
  ```

  NOTE: if you haven't used ChefDK for a while it's better to reinstall it -
        it looks like it's not updated properly by `brew upgrade`.

- create `devops` user on remote node

  ```sh
  (ws)$ ssh root@<remote-ip>
  (remote)# useradd devops -m -s /bin/bash -G sudo
  (remote)# passwd devops
  (remote)# exit
  ```

- don't create application user (say, `billing`) now

  application user should be created by application cookbook.

  <https://github.com/edeliver/edeliver/blob/master/libexec/app_config>:

  > Each app should run under its own system user.

- add new SSH host to _~./ssh/config_

  host itself and user must be equal to the name of application you're going
  to deploy on that host (`billing` in this case). if it's necessary to deploy
  another application on the same host create a separate SSH host named as that
  new application.

  to login as `devops` specify SSH user explicitly: `ssh devops@billing`.

  also when bootstrapping and converging SSH user (`devops`) is specified
  explicitly in _.chef/knife.rb_ config with `knife[:ssh_user]` option.

- don't add public keys to authorized keys files now

  public keys should be added to _~/.ssh/authorized_keys_ files of both
  application and `devops` users by `ssh_authorized_keys` cookbook.

- bootstrap remote node

  ```sh
  (ws)$ knife zero bootstrap billing --node-name billing
  / enter devops password for sudo command twice
  ```

  here first `billing` is a host name from SSH config.

  if you don't specify node name FQDN will be used by default -
  this is the name by which node is registered in a `chef-zero` server.

  NOTE: you cannot change node name by renaming node file and changing the name
        inside this file - node name is also stored in _/etc/chef/client.rb_ on
        remote node: if you change it locally new node file with the name from
        _/etc/chef/client.rb_ will be created after converging (with empty run_list).

- converge remote node

  ```sh
  (ws)$ knife node run_list add billing 'recipe[billing_app]'
  (ws)$ knife node environment_set billing staging
  (ws)$ berks vendor
  (ws)$ knife zero converge 'name:billing'
  / enter devops password for sudo command twice (on first converge only)
  ```

  berks vendors cookbooks to _berks-cookbooks/_ by default (both community
  and application cookbooks) and knife when converging searches exactly in
  this directory as specified in _.chef/knife.rb_:

  ```ruby
  cookbook_path ['berks-cookbooks']
  ```

- update attribute whitelist (if necessary)

  <https://github.com/higanworks/knife-zero/issues/99>

  - update corresponding attribute whitelist in _.chef/knife.rb_
  - update changes to _/etc/chef/client.rb_ on remote node:

    ```sh
    (ws)$ knife zero bootstrap billing --node-name billing --no-converge
    ```

    without `--no-converge` option command would overwrite node file.
