---
layout: post
title: chef-zero quickref
date: 2016-08-31 14:32:47 +0300
access: public
categories: [chef, chef-zero, knife-zero]
---

refined and focused quickref for chef-zero and knife-zero.

<!-- more -->

- install chefdk and knife-zero on workstation:

  ```sh
  (ws)$ brew update
  (ws)$ brew cask install chefdk
  (ws)$ chef gem install knife-zero
  ```

- configure devops and application (say, `builder`) users on remote node:

  ```sh
  (ws)$ ssh root@<remote-ip>
  (remote)# useradd devops -m -s /bin/bash -G sudo
  (remote)# passwd devops
  (remote)# useradd builder -m -s /bin/bash -G sudo
  (remote)# passwd builder
  (remote)# exit
  ```

- add new ssh host to _~./ssh/config_

  host itself and user must be equal to the name of application you're going
  to deploy on that host (`builder` in this case). if it's necessary to deploy
  another application on the same host create a separate ssh host named as that
  new application.

  to login as devops specify ssh user explicitly: `ssh devops@builder`.

  also when bootstrapping and converging ssh user (devops) is specified
  explicitly in _.chef/knife.rb_ with `knife[:ssh_user]` option.

- add public keys of devops and application users to authorized keys on remote node

  ```sh
  (ws)$ ssh devops@builder
  / enter devops password
  (remote)$ mkdir ~/.ssh
  (remote)$ vi ~/.ssh/authorized_keys
  / paste your public key (say, ~/.ssh/id_rsa.pub)
  (remote)$ exit
  ```

  the same steps for application user.

  now you can login to remote node without password.

- bootstrap remote node:

  ```sh
  (ws)$ knife zero bootstrap builder --node-name builder
  / enter devops password for sudo command
  ```

  here first `builder` is a host name from ssh config.

  if you don't specify node name FQDN will be used by default -
  this is the name by which node is registered in a chef-zero server.

  NOTE: you cannot change node name by renaming node file and changing the name
  inside this file - node name is also stored in _/etc/chef/client.rb_ on remote
  node: if you change it locally new node file with the name from
  _/etc/chef/client.rb_ will be created after converging (with empty run_list).

- converge remote node:

  ```sh
  (ws)$ knife node run_list add builder 'recipe[builder_app]'
  (ws)$ knife node environment_set builder staging
  (ws)$ berks vendor
  (ws)$ knife zero converge 'name:builder'
  / enter devops password for sudo command (on first converge only)
  ```

  berks vendors cookbooks to _berks-cookbooks/_ by default (both community
  and application cookbooks) and knife when converging searches exactly in
  this directory as specified in _.chef/knife.rb_:

  ```ruby
  cookbook_path ['berks-cookbooks']
  ```

- update attribute whitelist:

  <https://github.com/higanworks/knife-zero/issues/99>

  - update corresponding attribute whitelist in _.chef/knife.rb_
  - update changes to _/etc/chef/client.rb_ on remote node:

    ```sh
    (ws)$ knife zero bootstrap builder --node-name builder --no-converge 
    ```

    without `--no-converge` option command would overwrite node file.
