---
layout: post
title: chef-zero quickref
date: 2016-08-31 14:32:47 +0300
access: public
categories: [chef, chef-zero, knife-zero]
---

- install chefdk and knife-zero on workstation:

  ```sh
  (ws)$ brew cask update
  (ws)$ brew cask install chefdk
  (ws)$ chef gem install knife-zero
  ```

- configure devops user on remote node:

  ```sh
  (ws)$ ssh root@<remote-ip>
  (remote)# useradd devops -m -s /bin/bash -G sudo
  (remote)# passwd devops
  (remote)# exit
  ```

  add new ssh host (say, builder) to _~./ssh/config_ with devops user
  and remote ip address on workstation.

  ```sh
  (ws)$ ssh devops
  / enter devops password
  (remote)$ mkdir ~/.ssh
  (remote)$ vi ~/.ssh/authorized_keys
  / paste your public key (say, ~/.ssh/id_rsa.pub)
  (remote)$ exit
  ```

  now you can login to remote node without password.

- bootstrap remote node:

  ```sh
  (ws)$ knife zero bootstrap builder --node-name builder
  / enter devops password for sudo command
  ```

  if you don't specify node name FQDN will be used by default -
  this is the name by which node is registered in chef-zero server.
