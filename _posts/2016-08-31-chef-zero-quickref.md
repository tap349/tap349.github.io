---
layout: post
title: chef-zero quickref
date: 2016-08-31 14:32:47 +0300
access: public
categories: [chef, chef-zero, knife-zero]
---

- install chefdk and knife-zero on local machine:

  ```sh
  (local)$ brew cask update
  (local)$ brew cask install chefdk
  (local)$ chef gem install knife-zero
  ```

- configure devops user on remote node:

  ```sh
  (local)$ ssh root@<remote-ip>
  (remote)# useradd devops -m -s /bin/bash -G sudo
  (remote)# passwd devops
  (remote)# exit
  ```

  add new ssh host (say, builder) to _~./ssh/config_ with devops user
  and remote ip address on local machine.

  ```sh
  (local)$ ssh devops
  / enter devops password
  (remote)$ mkdir ~/.ssh
  (remote)$ vi ~/.ssh/authorized_keys
  / paste your public key (say, ~/.ssh/id_rsa.pub)
  (remote)$ exit
  ```

  now you can login to remote node without password.

- bootstrap remote node:

  ```sh
  (local)$ knife zero bootstrap builder --node-name builder
  / enter devops password for sudo command
  ```

  if you don't specify node name FQDN will be used by default -
  this is the name by which node is registered in chef-zero server.
