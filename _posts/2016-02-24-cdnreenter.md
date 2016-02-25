---
layout: post
title: cdnreenter
date: 2016-02-24 20:41:00 +0300
access: public
categories: [chef]
---

### undo knife-zero bootstrap run

- remove /etc/chef/ directory on node
- remove clients/ and nodes/ in chef repo
- bootstrap again

### log

.chef/knife.rb:

```ruby
# -----------------------------------------------------------------------------
# https://docs.chef.io/config_rb_knife_optional_settings.html
# -----------------------------------------------------------------------------

# same as adding -z option
local_mode true

# use ipaddress to connect to node (by default - fqdn)
knife[:attribute] = 'ipaddress'
# execute bootstrap operation with sudo
knife[:use_sudo] = true
enife[:ssh_user] = 'devops'
```

```sh
$ knife zero bootstrap cdnreenter --node-name cdnreenter
```

host and username for ssh are provided by ssh config (`cdnreenter` entry).

still it's necessary to enter devops password because bootstrap operation is run with sudo.

don't forget to supply node name with `--node-name cdnreenter` -
otherwise node name will be set to FQDN of the node
(`li1377-81.members.linode.com` in my case - as collected by ohai).

**NOTE**: FQDN (`li1377-81.members.linode.com`) will still be shown in
knife zero log when converging.

### cooking

```sh
$ berks vendor
$ knife zero converge 'name:cdnreenter'
```

set environment before converging any cookbooks using it
(almost any cookbooks that install anything inside app directory):

```sh
$ knife environment create production
$ knife node environment_set cdnreenter production
```

the 1st command would create production environment JSON file in environments/
directory (or else create it manually).

when boostraping with knife zero IP address is retrieved from ssh config -
later on it is stored somewhere in chef-zero server (that is it's no longer
read from either node JSON file or ssh config).
