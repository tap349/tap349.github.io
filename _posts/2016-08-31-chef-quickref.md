---
layout: post
title: Chef - Quickref (using chef-zero and knife-zero)
date: 2016-08-31 14:32:47 +0300
access: public
comments: true
categories: [chef, knife-zero]
---

refined and focused quickref for `chef-zero` and `knife-zero`.

<!-- more -->

* TOC
{:toc}
<hr>

- ws - workstation (local machine)

install Chef on workstation
---------------------------

use one of 2 options:

- install ChefDK

  ChefDK includes all required gems except for `knife-zero`:

  ```sh
  (ws)$ brew update
  (ws)$ brew cask reinstall chefdk
  (ws)$ chef gem install knife-zero
  ```

  if you haven't used ChefDK for a while it's better to reinstall it -
  it looks like it's not updated properly by `brew upgrade`.

- *[RECOMMENDED]* install gems manually

  create _Gemfile_:

  ```ruby
  source 'https://rubygems.org'

  gem 'berkshelf', '6.3.1'
  gem 'chef', '13.5.3'
  gem 'knife-zero', '1.19.3'
  ```

  ```sh
  $ bundle
  ```

create `devops` user on remote node
-----------------------------------

```sh
(ws)$ ssh root@<remote-ip>
(remote)# useradd devops -m -s /bin/bash -G sudo
(remote)# passwd devops
(remote)# exit
```

don't create application user (say, `billing`) now
--------------------------------------------------

application user should be created by application cookbook.

<https://github.com/edeliver/edeliver/blob/master/libexec/app_config>:

> Each app should run under its own system user.

add new SSH host to _~./ssh/config_
-----------------------------------

host itself and user must be equal to the name of application you're going
to deploy on that host (`billing` here). if it's necessary to deploy another
application on the same host create a separate SSH host with the name of new
application.

_~/.ssh/config_:

```
Host billing
 User billing
 Hostname LINODE_IP
 IdentityFile ~/.ssh/id_rsa
 ForwardAgent yes
```

to login as `devops` user specify him explicitly: `ssh devops@billing`
(don't use `devops` user in SSH config entry so that application user
is used by default).

also when bootstrapping and converging SSH user (`devops`) is specified
explicitly in _.chef/knife.rb_ config with `knife[:ssh_user]` option.

don't add public keys to authorized keys files now
--------------------------------------------------

public keys should be added to _~/.ssh/authorized_keys_ files of both
application and `devops` users by `ssh_authorized_keys` cookbook.

bootstrap remote node
---------------------

```sh
(ws)$ knife zero bootstrap billing --node-name billing
/ enter devops password twice (for login and sudo command)
```

for login and running `sudo` command.

here first `billing` is a host name from SSH config.

if you don't specify node name explicitly, FQDN will be used by default -
this is the name by which node is registered in a `chef-zero` server.

2 files will be created after bootstrapping:

- _clients/billing.json_ (contains public key)
- _nodes/billing.json_ (contains automatic whitelisted attributes now)

NOTE: you cannot change node name by renaming node file and changing the name
      inside this file - node name is also stored in _/etc/chef/client.rb_ on
      remote node: if you change it locally new node file with the name from
      _/etc/chef/client.rb_ will be created after converging (its `run_list`
      will be empty).

create production environment
-----------------------------

```sh
(ws)$ knife environment create production
```

_environments/production.json_ file will be created with the following
content:

```json
{
  "name": "production"
}
```

this file can be created manually of course.

it doesn't matter much what the name of production environment is -
be it `production` or `prod`. these names are mostly used internally
when passing attributes for different environments between cookbooks.

still stick to `prod` name when you are going to create application
cookbook for Phoenix project inside this Chef repo and `production`
- for Rails project: environment name might be interpolated inside
config templates so it's better to use native environment names to
avoid confusion.

add application cookbook default recipe to node run list
--------------------------------------------------------

```sh
(ws)$ knife node run_list add billing 'recipe[app_billing]'
```

converge remote node
--------------------

```sh
(ws)$ knife node environment_set billing prod
(ws)$ berks vendor
(ws)$ knife zero converge 'name:billing'
/ enter devops password twice (on first converge only)
```

`berks` vendors cookbooks to _berks-cookbooks/_ by default (both community
and application cookbooks) and `knife` when converging searches exactly in
this directory as specified in _.chef/knife.rb_:

```ruby
cookbook_path ['berks-cookbooks']
```

update attribute whitelist (if necessary)
-----------------------------------------

1. <https://github.com/higanworks/knife-zero/issues/99>

- update corresponding attribute whitelist in _.chef/knife.rb_
- update changes to _/etc/chef/client.rb_ on remote node:

  ```sh
  (ws)$ knife zero bootstrap billing --node-name billing --no-converge
  ```

  command would overwrite node file without `--no-converge` option.
