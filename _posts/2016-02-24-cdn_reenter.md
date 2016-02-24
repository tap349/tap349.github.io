---
layout: post
title: cdn_reenter
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

# execute bootstrap operation with sudo
knife[:use_sudo] = true
knife[:ssh_user] = 'devops'
```

```sh
$ knife zero bootstrap cdn_reenter --node-name cdn_reenter
```

host and username for ssh are provided by ssh config (`cdn_reenter` entry).

still it's necessary to enter devops password because bootstrap operation is run with sudo.

don't forget to supply node name with `--node-name cdn_reenter` -
otherwise node name will be set to FQDN of the node
(`li1377-81.members.linode.com` in my case - as collected by ohai).

**NOTE**: FQDN (`li1377-81.members.linode.com`) will still be shown in
knife zero log when converging.
