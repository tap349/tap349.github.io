---
layout: post
title: cdnreenter
date: 2016-02-24 20:41:00 +0300
access: public
categories: [chef]
---

actions log to provision cdnreenter.

<!-- more -->

### undo knife-zero bootstrap run

- remove _/etc/chef/_ directory on node
- remove _clients/_ and _nodes/_ in chef repo
- bootstrap again

### knife configuration

_.chef/knife.rb_:

```ruby
# -----------------------------------------------------------------------------
# https://docs.chef.io/config_rb_knife_optional_settings.html
# -----------------------------------------------------------------------------

# same as adding -z option
local_mode true

# use ipaddress to connect to node (FQDN is used by default)
knife[:attribute] = 'ipaddress'
# execute bootstrap operation with sudo
knife[:use_sudo] = true
knife[:ssh_user] = 'devops'

cookbook_path ['berks-cookbooks']
```

by default FQDN is used to connect to node but it may be blank after
bootstrapping node and creating new node in chef-zero server.
thus it's better to always use IP address instead of FQDN by setting
knife's `attribute` option to `ipaddress` (this is automatic node attribute
collected by ohai).
after boostrapping node IP address is stored somewhere in
chef-zero server configuration - it can be viewed by running
`knife node show cdnreenter`. it's no longer read from either node file or
ssh config but from chef-zero server configuration.
node file must have the same name as node name set when bootstrapping the node
and later stored in chef-zero configuration.

### bootstrapping the node

```sh
$ knife zero bootstrap cdnreenter --node-name cdnreenter
```

host and username for ssh are provided by ssh config (`cdnreenter` entry).
TODO: get it clear whether it's possible to bootstrap node without setting
`ssh_user` option for knife in _.chef/knife.rb_.

still it's necessary to enter devops password because bootstrap operation
is run with sudo.

don't forget to supply node name with `--node-name cdnreenter` -
otherwise node name will be set to FQDN of the node (as collected by ohai).

### converging the node

set environment before converging any cookbooks using it
(almost any cookbooks that install anything inside app directory):

```sh
$ knife environment create production
$ knife node environment_set cdnreenter production
```

the 1st command would create production environment JSON file in environments/
directory (or else create it manually).

```sh
$ berks vendor
$ knife zero converge 'name:cdnreenter'
```

when boostraping with knife zero IP address is retrieved from ssh config -
later on it is stored somewhere in chef-zero server (that is it's no longer
read from either node JSON file or ssh config).
