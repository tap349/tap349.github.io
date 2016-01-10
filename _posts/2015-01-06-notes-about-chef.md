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

1. chef node (deployment target - this is what configured, has chef-client or chef-solo installed)
2. chef server (combined with chef node using chef-solo)
3. workstation (from which knife or knife-solo is run - can be remote machine as well)

#### core principles:

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
## [Chef Solo](https://docs.chef.io/chef_solo.html)
___

**chef-solo**

- open-source version of chef-client with limited functionality
- doesn't require access to chef server
- requires that a cookbook be on the same physical disk as the node
- doesn't support authentication and authorization

**knife**

- provides interface between local chef-repo and chef server

**knife-solo**

- installs knife as dependency
- adds 5 subcommands to knife:
  - `knife solo init` (create a new kitchen)
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
- uploaded to chef server (or chef node when using chef-solo)

create a new kitchen in current directory:

```sh
$ knife solo init .
```

file structure:

directory or file   | usage
--------------------|-----------------------------------------------------------
_.chef/_            | stores .pem files and _knife.rb_
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

### [VENDOR COOKBOOKS](https://docs.chef.io/cookbooks.html)

_Berksfile_:

```
cookbook 'apache2'
```

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

### [NODES](https://docs.chef.io/nodes.html)

**NOTE**: probably nodes are configured in node files for chef-solo only -
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

### [ATTRIBUTES](https://docs.chef.io/attributes.html)

**attribute**

- specific detail about node
- provided from the following locations:
  - node files
  - attribute files (in cookbook)
  - recipes (in cookbook)
  - roles
  - environments

![attribute precendence table](https://docs.chef.io/_images/overview_chef_attributes_table.png)

normal attributes:

- attributes in node file are normal attributes
- they are not reset before each chef-client run unlike other attributes

accessor methods are automatically defined for attributes in Ruby files:

```ruby
default['apache']['dir'] = '/etc/apache2'
default.apache.dir = '/etc/apache2'
```

### [ROLES](https://docs.chef.io/roles.html)

**role**

- server template (web, database, etc.)
- contains server specific run-list and attributes
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
`run_list`            | yes       | role specific run-list (may include other roles)
`env_run_lists`       |           | environment specific run-lists

use role in node file:

```yaml
{
  run_list: [
    'role[web]'
  ]
}
```

### [ENVIRONMENTS](https://docs.chef.io/environments.html)

- environment template (development, staging, production, etc.)
- contains environment specific attributes
- doesn't have run-list - attributes only
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

### [DATA BAGS](https://docs.chef.io/data_bags.html)

**data bag**

- global variable stored as JSON data
- often used to store sensitive data (credentials, etc.)
- data bag consists of data bag items: _DATA\_BAG/DATA\_BAG\_ITEM.json_

structure (according to official docs data bag item should contain only the contents of `raw_data`):

key                   | required? | description
----------------------|:---------:|---------------------------------------------
`name`                | yes       | unique name (usually the same as file name)
`chef_type`           | yes       | 'data_bag_item'
`json_class`          | yes       | 'Chef::DataBagItem'
`data_bag`            | yes       | data bag name
`raw_data`            | yes       | attributes of data bag item (`id` attribute is required)

___
## [SITE COOKBOOK](https://docs.chef.io/cookbooks.html)
___

- fundamential unit of configuration and policy distribution

create custom cookbook:

```sh
$ cd site-cookbooks/
$ knife cookbook create COOKBOOK
```

structure:

directory or file |
------------------|
_attributes/_     |
_definitions/_    |
_files/_          |
_libraries/_      |
_providers/_      |
_recipes/_        |
_resources/_      |
_templates/_      |
_metadata.rb_     |

### [RECIPES](https://docs.chef.io/recipes.html)

**recipe**

- the most fundamental configuration element
- written in Ruby
- stored in cookbook
- consists of resources
- may be included in other recipes
- may depend on other recipes
- must be added to run-list before it can be used by chef-client

default recipe - _default.rb_.

#### include other recipes

recipe can include other recipes from other cookbooks using `include_recipe` method:

```ruby
include_recipe 'apache2::mod_ssl'
```

included recipe must be declared as dependency in _metadata.rb_:

```ruby
depends 'apache2'
```

- included recipe is just inlined at the point where it's included
- only the first inclusion within recipe is processed - subsequent ones are ignored

#### write to log from within a recipe

log levels:

- debug
- info
- warn
- error
- fatal

```ruby
Chef::Log.info 'some useful information'
```

### [RESOURCES](https://docs.chef.io/resource.html)

**resource**

- describes desired state for configuration item
- declares steps required to bring configuration item to desired state
- resources are grouped into recipes
- during chef-client run each resource is associated with **provider** (platform specific)
- provider does actual job described in the resource

has:

- type (package, template, service, etc.)
- name

consists of:

- attributes (one or more) - aka properties
- actions (one or more)

syntax:

```ruby
type 'name' do
  attribute 'value'
  action :type_of_action
end
```

- most properties have default values
- all actions have default values

only non-default properties and actions must be specified.

platform resources (available from chef-client directly and don't require a cookbook):

resource        | description
----------------|---------------------------------------------------------------
`script`        | execute script using specified interpretor
`bash`          | execute script using Bash interpretor
`ruby`          | execute script using Ruby interpretor
`link`          | create sym or hard links
`directory`     | manage directories
`cookbook_file` | transfer files from subdirectory (_PLATFORM_ or _default_ for any platform) of _files/_
`template`      | transfer files from subdirectory of _templates/_ (i.e. static files generated from ERB templates)
`gem_package`   | install gem system-wide
`chef_gem`      | install gem into the instance of Ruby dedicated to chef-client (can be required immediatelly after it's installed)
`cron`          | modify cron entries
`user`          | manage users

**NOTE**: for all resources dealing with files (`directory`, `cookbook_file`, etc.)
          path on chef node is specified as resource name.

sample resource:

```ruby
directory '/tmp/something' do
  owner 'root'
  group 'root'
  mode 00755
  action :create
end
```

### [ATTRIBUTE FILES](https://docs.chef.io/attributes.html)

- when cookbook is run against a node attributes
  inside **all** attribute files are evaluated in the context of node object
- cookbook attributes are usually placed into
  _attributes/default.rb_ -
  file name _default.rb_ is just a convention here.
  if necessary attributes can be grouped into several
  attribute files with arbitrary names

sample attribute file:

```ruby
default['apache']['dir']          = '/etc/apache2'
default['apache']['listen_ports'] = [ '80','443' ]
```

use of node object (`node`) is implicit here - it can be specified explicitly:

```ruby
node.default['apache']['dir']          = '/etc/apache2'
node.default['apache']['listen_ports'] = [ '80','443' ]
```

as mentioned above dot syntax can be used instead of hash keys:

```ruby
default.apache.dir          = '/etc/apache2'
default.apache.listen_ports = [ '80','443' ]
```

`default` is attribute type here - it has nothing to do with attribute file name.
it's also possible to use other attribute types:

```ruby
# override
override['apache']['dir'] = '/etc/apache2'

# normal
set['apache']['dir']    = '/etc/apache2'
normal['apache']['dir'] = '/etc/apache2' # set is an alias of normal
```

**NOTE**: in recipe and node file use of node object must be explicit -
          these files are not evaluated in the context of node object
          like attribute files!

```ruby
node.set['apache']['dir']    = '/etc/apache2'
node.normal['apache']['dir'] = '/etc/apache2' # same as above
node['apache']['dir']        = '/etc/apache2' # same as above
```

### [METADATA.RB](https://docs.chef.io/cookbook_repo.html)

- lives at the top of each cookbook's directory
- provides hints to chef server to deploy cookbook correctly

setting            | description
-------------------|------------------------------------------------------------
`name`             | cookbook name (main setting)
`maintainer`       |
`maintainer_email` |
`description`      |
`long_description` | usually the contents of README.md
`version`          | three-number version sequence
`chef_version`     | range of chef-client versions supported by cookbook
`attribute`        | attribute required to configure cookbook
`provides`         | recipe or resource provided by cookbook (auxilliary)
`recipe`           | description for recipe (cosmetic value)
`depends`          | cookbook dependency on another cookbook
`conflicts`        | FIO. conflicting cookbook or cookbook version
`recommends`       | FIO. recommended cookbook
`suggests`         | FIO. suggested cookbook (weaker than `recommends`)
`replaces`         | FIO. cookbook to be replaced by this cookbook
`supports`         | supported platform

