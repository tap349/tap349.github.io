---
layout: post
title: Chef - How knife connects to node
date: 2016-02-27 01:16:00 +0300
access: public
comments: true
categories: [chef]
---

how knife-solo and knife-zero connect to the node when bootstrapping and cooking.

<!-- more -->

knife-solo
----------

### bootstrapping

it's possible to connect to node using:

- *domain or IP address*

  current user is used unless it's specified explicitly
  (user@domain or user@ip).

  you are always prompted to enter password.

- *host from SSH config*

  passwordless authentication is used.

- *user@ip*

  passwordless authentication is used if
  corresponding entry in SSH config is found.

  user can be specified with `knife[:ssh_user]` option in _knife.rb_.

knife-solo creates new node file in _nodes/_ after bootstrapping the node.

node name (as listed in `knife node list -z` and used later for cooking)
and node file name (file in _nodes/_) are the same and
they depend on what was used when bootstrapping the node:

  - domain (when using domain or host from SSH config)
  - IP address (when using IP address).

### cooking

when using knife-solo and chef-solo there is no chef server where
information about nodes might be stored.
that is why knife connects to node in the same way as described above
for bootstrapping.

cooking itself:

- chef repo is uploaded to _~/chef-solo/_ directory in
  specified user's home directory
- chef-solo searches for node file in uploaded chef repo -
  node file name must be the same as domain or IP address used
  to connect to the node. if such node file is not found
  chef-solo creates a new one with empty run-list
- chef-solo provisions the node using node file
  (it is created if it doesn't exist - see previous step)

it's possible to specify alternate node file whose name differs from
domain or IP address used to connect to the node:

```sh
$ knife solo cook tap349 nodes/tap349-2.json
```

knife-zero
----------

### bootstrapping

same as for knife-solo except that it's possible to specify node name
explicitly with `--node-name` option when bootstrapping the node -
node and node file will both have specified name.

### cooking

after bootstrapping node is added to the list of nodes on chef-zero
server (use `knife zero show tap349 -z` to view node information).

information about the node (such as FQDN, IP, Platform, etc.) is retrieved
from automatic attributes collected by ohai during bootstrapping
(corresponding attributes: fqdn, ipaddress, platform, etc.).

when cooking we **search** for node on chef-zero server using node name
assigned to the node during bootstrapping:

```sh
$ knife zero converge 'name:tap349' -z
```

when cooking knife-zero tries to connect to node using FQDN by default
(node's FQDN is read from node information on chef-zero server) -
consequently changing host, domain or IP address in
either SSH config or node file itself has no effect on cooking.

it's possible to make knife-zero use IP instead of FQDN for opening
the connection to the node by specifying `-a, --attribute` option
([doc](https://knife-zero.github.io/30_subcommands/)):

> The attribute to use for opening the connection

```sh
$ knife zero converge 'name:tap349' -a ipaddress -z
```

or set attribute permanently in _knife.rb_:

```ruby
knife[:attribute] = 'ipaddress'
```

BTW after setting specific attribute for knife only this attribute
will be shown in node information with `knife node show tap349 -z` command.
also it's possible to specify multiple attributes for knife in
`knife[:attribute]` option using array `['ipaddress', 'fqdn']`:
only specified attributes will be shown in node information but it will
break `knife zero converge` command as it expects a string - not array:

```sh
$ knife zero converge 'name:tap349' -z
...
Exception: NoMethodError: undefined method `split' for ["ipaddress"]:Array
```

on the other hand when searching for nodes the value of
`knife[:attribute]` option must be set to array:

```sh
$ knife search node "name:tap349" -F json -z
...
Exception: NoMethodError: undefined method `each' for "ipaddress":String
```

it's all really confusing :) the point might be that knife-zero is a
3rd-party gem not officially supported by chef and it hasn't been tested
with all available knife options.
