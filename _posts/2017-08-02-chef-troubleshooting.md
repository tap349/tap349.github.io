---
layout: post
title: Chef - Troubleshooting
date: 2017-08-02 18:25:32 +0300
access: public
categories: [chef]
---

<!-- more -->

* TOC
{:toc}
<hr>

## installation of both knife-solo and knife-zero

`knife` executables from both gems might conflict since `knife-solo` is
installed as ordinary gem (`gem install knife-solo` - _~/.rbenv/shims/knife_)
while `knife-zero` is installed using ChefDK (`chef install gem knife-zero` -
_/opt/chefdk/bin/knife_) 

so only one of them might be available at any given time -
see [rbenv]({% post_url 2016-03-30-rbenv %}) for details.

## Doing old-style registration with the validation key at

```sh
$ knife zero bootstrap builder --node-name builder
Doing old-style registration with the validation key at ...
Delete your validation key in order to use your user credentials instead
```

**solution**

<https://github.com/higanworks/knife-zero/issues/24>

> Don't worry about it.
> This warning message is not effect Knife-Zero's behavior.
> Because authrization of Chef-Zero is dummy.

## ArgumentErrorwrong number of arguments (given 1, expected 2)

```sh
$ knife zero bootstrap billing --node-name billing

Connecting to billing
ERROR: ArgumentErrorwrong number of arguments (given 1, expected 2)
ERROR: /opt/chefdk/embedded/lib/ruby/gems/2.3.0/gems/net-ssh-3.2.0/lib/net/ssh/connection/keepalive.rb:36:in `send_as_needed'
```

**solution**

it seems like `knife` is using old Ruby version provided by ChefDK.

```sh
$ brew cask reinstall chefdk
$ chef gem install knife-zero
```

## There is a dependency conflict, but the solver could not determine the precise cause in the time allotted

```sh
$ berks vendor
Resolving cookbook dependencies...
...
Fetching cookbook index from https://supermarket.chef.io...
There is a dependency conflict, but the solver could not determine the precise cause in the time allotted.
```

**solution**

error occurs when you add non-existing custom cookbook as dependency in your
application cookbook's _metadata.rb_ file - so just remove that dependency.
