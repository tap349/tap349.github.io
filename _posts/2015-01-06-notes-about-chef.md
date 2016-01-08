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

___
### [Chef Solo](https://docs.chef.io/chef_solo.html)
___

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

```sh
$ knife solo init .
```

directory or file   | usage
--------------------|-----------------------------------------------------------
_.chef/_            | stores .pem files and knife.rb
_cookbooks/_        | vendor cookbooks installed with berkshelf
_data\_bags/_       | data bags
_environments/_     | environments
_nodes/_            | nodes
_roles/_            | roles
_site-cookbooks/_   | custom cookbooks
_Berksfile_         | like Gemfile for rubygems

_.chef/knife.rb_

- chef-repo specific knife settings (primarily paths to cookbooks, node files, etc.)
- loaded every time knife is run

#### COOKBOOKS

_Berksfile_

    cookbook 'apache2'

install cookbook globally into _~/.bershelf/cookbooks/_:

```sh
$ berks install
```

install cookbook both locally into _cookbooks/_ and
globally into _~/.bershelf/cookbooks/_:

```sh
$ berks vendor cookbooks
```

**NOTE**: command `berks install --path cookbooks` does the same but is deprecated.

it's necessary to specify directory (_cookbooks/_) -
otherwise berkshelf will install cookbook into _berks-cookbooks/_ directory.

`knife[:berkshelf_path]` option has no effect on where berkshelf installs cookbooks
 - most likely it's used by knife to search for cookbooks while provisioning.
but since `berks vendor` command installs cookbook into _~/.bershelf/cookbooks/_
as well I can't figure out how you might end up having cookbook installed locally only
(which would justify existence of `knife[:berkshelf_path]` option).

#### [NODES](https://docs.chef.io/nodes.html)

NOTE: probably nodes are configured in node files for chef-solo only -
chef-client interacts with chef server to retrieve node configuration.

**node**

- represents machine (physical, virtual, etc.)
- create separate JSON file for each node in _nodes/_
- node file name (as a rule): _DOMAIN.json_

`run_list`

- main key in node file
- contains array of recipes and roles
- recipe name format (2 variants):
  - `COOKBOOK::RECIPE`
  - `recipe[COOKBOOK::RECIPE]`
- _default.rb_ - default recipe of cookbook
  - executed when only cookbook name is specified without specific recipe
  - can be explicitly called with `COOKBOOK::default`

sample node file:

```yaml
{
  run_list: [
    'recipe[apache2]'
  ]
}
```

#### [ROLES](https://docs.chef.io/roles.html)

**role**

- server template (web, database, etc.)
- contains server specific run list and attributes
- create separate JSON file for each role in _roles/_

structure:

key                   | required? | description
----------------------|:---------:|---------------------------------------------
`name`                | yes       | unique name (usually the same as file name)
`description`         | yes       |
`chef_type`           | yes       | 'role'
`json_class`          | yes       | 'Chef::Role'
`default_attributes`  |           | role specific attributes, can be overidden in node files
`override_attributes` |           | role specific forced attributes, cannot be overidden in node files
`run_list`            | yes       | role specific run list (may include other roles)
`env_run_lists`       |           | environment specific run lists

use role in node file:

```yaml
{
  run_list: [
    'role[web]'
  ]
}
```

#### [ATTRIBUTES](https://docs.chef.io/attributes.html)

**attribute**

- specific detail about node
- defined in:
  - node files
  - cookbooks (attribute files or recipes)
  - roles
  - environments

![attribute precendence table](https://docs.chef.io/_images/overview_chef_attributes_table.png)

NOTE: attributes in node file are normal attributes.

#### [ENVIRONMENTS](https://docs.chef.io/environments.html)

- environment template (development, staging, production, etc.)
- contains environment specific attributes
- doesn't have run list - attributes only
- create separate JSON file for each environment in _environments/_
- default environment is `_default` - all nodes are placed there
  unless another environment is specified
- each node can be in exactly one environment

structure:

key                   | required? | description
----------------------|:---------:|---------------------------------------------
`name`                | yes       | unique name (usually the same as file name)
`description`         | yes       |
`chef_type`           | yes       | 'environment'
`json_class`          | yes       | 'Chef::Environment'
`default_attributes`  |           | environment specific attributes, can be overidden in node files
`override_attributes` |           | environment specific forced attributes, cannot be overidden in node files

set environment in node file:

```yaml
{
  'run_list': [
    'recipe[apache2]'
  ],
  'chef_environment': 'development'
}
```

or else environment can be applied using `-E` knife argument:

```sh
$ knife solo cook -E development
```

#### [DATA BAGS](https://docs.chef.io/data_bags.html)

**data bag**

- global variable stored as JSON data
- data bag consists of data bag items: _DATA\_BAG/DATA\_BAG\_ITEM.json_

structure (according to official doc data bag item should contain only the contents of `raw_data`):

key                   | required? | description
----------------------|:---------:|---------------------------------------------
`name`                | yes       | unique name (usually the same as file name)
`chef_type`           | yes       | 'data_bag_item'
`json_class`          | yes       | 'Chef::DataBagItem'
`data_bag`            | yes       | data bag name
`raw_data`            | yes       | attributes of data bag item (`id` attribute is required)

___
### Cookbook
___


