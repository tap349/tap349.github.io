---
layout: post
title: notes about chef
date: 2016-01-06 00:42:00 +0300
access: public
categories: [chef]
---

summary from book 'Cooking infrastructure by Chef' by Alexey Vasiliev

<!-- more -->

participants:

1. chef node (this is what configured, has chef-client or chef-solo installed)
2. chef server (absent when using chef-solo)
3. workstation (from which knife or knife-solo is run)

##### core principles:

* *idempotence*
* *thick clients, thin server*
  <br>
  chef does as much work as possible on the node and as little as possible on the server
* *order matters*
  <br>
  within a recipe resources are applied in the order they are listed

**chef-client**

- installed on each node managed by chef server
- used to bring the node into the expected state

______
### Chef Solo
______

**chef-solo**

- open-source version of chef-client with limited functionality
- doesn't require access to server
- requires that a cookbook be on the same physical disk as the node
- doesn't support authentication and authorization

**knife**

- provides interface between local chef-repo and chef server

**knife-solo**

- installs knife as dependency
- adds 5 subcommands to knife:
  - `knife solo init`
  - `knife solo prepare` (install chef-solo on target host)
  - `knife solo cook` (upload current kitchen to target host and run chef-solo there)
  - `knife solo boostrap` (prepare + cook)
  - `knife solo clean` (remove uploaded kitchen from target host)

**berkshelf**

- manages cookbooks and their dependencies (like bundler for rubygems)
- librarian-chef - alternative

**kitchen**

- chef-repo
- located on workstation

```shell
$ knife solo init .
```

directory or file | usage
------------------|-------------------------------------------------------------
.chef/            | stores .pem files and knife.rb
cookbooks/        | vendor cookbooks installed with berkshelf
data_bags/        | data bags
environments/     | environments
nodes/            | nodes
roles/            | roles
site-cookbooks/   | custom cookbooks
Berksfile         | like Gemfile for rubygems

*`.chef/knife.rb`*

- chef-repo specific knife settings (primarily paths to cookbooks, nodes, etc.)
- loaded every time knife is run

#### install vendor cookbooks

==*`Berksfile`*==
```
cookbook 'apache2'
```

```shell
$ berks install
```

or just

```shell
$ berks
```

this will install cookbook into *`~/.bershelf/cookbooks/`*

to install cookbook inside chef-repo run:

```shell
$ berks vendor cookbooks
```

it's necessary to specify directory - otherwise berkshelf will install cookbook
into *`berks-cookbooks/`* directory.
