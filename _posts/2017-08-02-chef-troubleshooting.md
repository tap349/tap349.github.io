---
layout: post
title: Chef - Troubleshooting
date: 2017-08-02 18:25:32 +0300
access: public
comments: true
categories: [chef]
---

<!-- more -->

* TOC
{:toc}
<hr>

knife-solo and knife-zero are installed at the same time
--------------------------------------------------------

`knife` executables from these gems might conflict with each other since
`knife-solo` is installed as ordinary gem (`gem install knife-solo` →
_~/.rbenv/shims/knife_) while `knife-zero` is installed using ChefDK
(`chef install gem knife-zero` → _/opt/chefdk/bin/knife_).

so only one of them can be available at any given time -
see [rbenv]({% post_url 2016-03-30-rbenv %}) for details.

Doing old-style registration with the validation key at
-------------------------------------------------------

```sh
$ knife zero bootstrap builder --node-name builder
Doing old-style registration with the validation key at ...
Delete your validation key in order to use your user credentials instead
```

**solution**

<https://github.com/higanworks/knife-zero/issues/24>:

> Don't worry about it.
> This warning message is not effect Knife-Zero's behavior.
> Because authrization of Chef-Zero is dummy.

ArgumentErrorwrong number of arguments (given 1, expected 2)
------------------------------------------------------------

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

There is a dependency conflict, but the solver could not determine the precise cause in the time allotted
---------------------------------------------------------------------------------------------------------

```sh
$ berks vendor
Resolving cookbook dependencies...
...
Fetching cookbook index from https://supermarket.chef.io...
There is a dependency conflict, but the solver could not determine the precise cause in the time allotted.
```

**solution**

the error occurs after adding non-existing custom cookbook as dependency in
application cookbook's _metadata.rb_ file - so just remove that dependency.

undefined method `set' for Chef::Platform:Class
-----------------------------------------------

```
$ knife zero converge 'name:billing'
Compiling Cookbooks...

================================================================================
Recipe Compile Error in /var/chef/cache/cookbooks/git/libraries/z_provider_mapping.rb
================================================================================

NoMethodError
-------------
undefined method `set' for Chef::Platform:Class

Cookbook Trace:
---------------
  /var/chef/cache/cookbooks/git/libraries/z_provider_mapping.rb:6:in `<top (required)>'

Relevant File Content:
----------------------
/var/chef/cache/cookbooks/git/libraries/z_provider_mapping.rb:

  1:  # provider mappings for Chef 11
  2:
  3:  #########
  4:  # client
  5:  #########
  6>> Chef::Platform.set platform: :amazon, resource: :git_client, provider: Chef::Provider::GitClient::Package
  7:  Chef::Platform.set platform: :centos, resource: :git_client, provider: Chef::Provider::GitClient::Package
```

**solution**

1. <https://github.com/chef-cookbooks/git/issues/121>
2. <https://github.com/chef-cookbooks/git/issues/119>

```sh
$ berks update git
```

also I had to update all cookbooks that depend on `git` cookbook
which are used in my application cookbook:

```sh
$ berks update appbox
```

then remove directory with vendored cookbooks and vendor them again
(this is what really helped):

```sh
$ rm -rf berks-cookbooks/
$ berks vendor
```

uninitialized constant Chef::Resource::PostgresqlUser
-----------------------------------------------------

```
$ knife zero converge 'name:billing'
...
Recipe: postgresql::setup_users
  * postgresql_user[devops] action create

    ================================================================================
    Error executing action `create` on resource 'postgresql_user[devops]'
    ================================================================================

    NameError
    ---------
    uninitialized constant Chef::Resource::PostgresqlUser

    Cookbook Trace:
    ---------------
    /var/chef/cache/cookbooks/postgresql/providers/user.rb:54:in `load_current_resource'

    Resource Declaration:
    ---------------------
    # In /var/chef/cache/cookbooks/postgresql/recipes/setup_users.rb

      8:   postgresql_user user["username"] do
```

**solution**

1. <https://github.com/realchrisolin/chef-postgresql/commit/7683f11aedf967bca4c40eb8a72dc14d0f0ebf03>

it looks like new version of Chef doesn't provide `Chef::Resource::PostgresqlUser`
resource (same for `PostgresqlDatabase` and `PostgresqlExtension` resources).

_providers/user.rb_ (`chef-postgresql` cookbook):

```diff
- @current_resource = Chef::Resource::PostgresqlUser.new(new_resource.name)
+ @current_resource = Chef::Resource.resource_for_node(:postgresql_user, node).new(new_resource.name)
```

make the same changes in _providers/database.rb_ and _providers/extension.rb_.

undefined method `[]' for nil:NilClass (nginx/recipes/ohai_plugin.rb:27)
------------------------------------------------------------------------

```
$ knife zero converge 'name:billing'
...
Compiling Cookbooks...

================================================================================
Recipe Compile Error in /var/chef/cache/cookbooks/billing_app/recipes/default.rb
================================================================================

NoMethodError
-------------
undefined method `[]' for nil:NilClass

Cookbook Trace:
---------------
  /var/chef/cache/cookbooks/nginx/recipes/ohai_plugin.rb:27:in `from_file'
  /var/chef/cache/cookbooks/nginx/recipes/source.rb:40:in `from_file'
  /var/chef/cache/cookbooks/phoenix_nginx/recipes/default.rb:11:in `from_file'
```

**solution**

1. <https://github.com/miketheman/nginx/issues/419>

use [chef_nginx](https://github.com/chef-cookbooks/chef_nginx) cookbook instead.

`berks install` doesn't install dependency
------------------------------------------

_cookbooks/phoenix_nginx/metadata.rb_:

```ruby
depends 'chef_nginx'
```

but `berks install` doesn't install this cookbook even though it's available
in Chef supermarket.

**solution**

delete _Berksfile.lock_ and run `berks install` again.

Failed to restart nginx.service: Unit nginx.service not found.
--------------------------------------------------------------

```
$ knife zero converge 'name:billing'
...
* service[nginx] action restart

  ================================================================================
  Error executing action `restart` on resource 'service[nginx]'
  ================================================================================

  Mixlib::ShellOut::ShellCommandFailed
  ------------------------------------
  Expected process to exit with [0], but received '5'
  ---- Begin output of /bin/systemctl --system restart nginx ----
  STDOUT:
  STDERR: Failed to restart nginx.service: Unit nginx.service not found.
  ---- End output of /bin/systemctl --system restart nginx ----
  Ran /bin/systemctl --system restart nginx returned 5
```

**solution**

obviously `chef_nginx` cookbook has created init script but
for some reason is trying to restart systemd unit.

_cookbooks/phoenix_nginx/attributes/default.rb_:

```diff
- override['nginx']['init_style'] = 'init'
+ override['nginx']['init_style'] = 'systemd'
```

undefined method `definition' for Custom resource ruby_rbenv_ruby from cookbook ruby_rbenv
------------------------------------------------------------------------------------------

```
$ knife zero converge 'name:billing'
...
================================================================================
Recipe Compile Error in /var/chef/cache/cookbooks/app_billing/recipes/default.rb
================================================================================

NoMethodError
-------------
undefined method `definition' for Custom resource ruby_rbenv_ruby from cookbook ruby_rbenv

Cookbook Trace:
---------------
  /var/chef/cache/cookbooks/ruby_rbenv/recipes/user.rb:48:in `block (3 levels) in from_file'
```

**solution**

1. <https://github.com/sous-chefs/ruby_rbenv/issues/166>

this error occurs after updating `ruby_rbenv` cookbook to its latest version
as of now - 2.0.6. so it seems to be a bug in `ruby_rbenv` cookbook itself -
downgrade it to some earlier version.

_cookbooks/rails_rbenv/metadata.rb_:

```ruby
depends 'ruby_rbenv', '1.0.1'
```

```sh
$ berks update ruby_rbenv
$ berks vendor && knife zero converge 'name:billing'
```

1 node found, but does not have the required attribute to establish the connection
----------------------------------------------------------------------------------

```sh
$ knife zero converge 'name:billing'
...
FATAL: 1 node found, but does not have the required attribute to establish the connection.
Try setting another attribute to open the connection using --attribute.
```

**solution**

_Gemfile_:

```ruby
gem 'berkshelf', '7.0.2'
gem 'chef', '14.1.1'
gem 'knife-zero', '1.19.3'
```

this problem is caused by new version of `knife` which doesn't understand
`knife[:attribute]` setting in _.chef/knife.rb_ any more.

<https://docs.chef.io/config_rb_knife.html>:

> knife[:ssh_attribute]
>   The attribute used when opening an SSH connection.

so either specify `-a` option on command line:

```sh
$ knife zero converge 'name:billing' -a ipaddress
```

or use new `ssh_attribute` setting in _.chef/knife.rb_:

```diff
- knife[:attribute] = 'ipaddress'
+ knife[:ssh_attribute] = 'ipaddress'
```

see also discussion on `ipaddress` vs. `['ipaddress']` in
[Chef - How knife connects to node]({% post_url 2016-02-27-chef-how-knife-connects-to-node %}).

fatal: could not read Username for 'https://github.com': Device not configured
------------------------------------------------------------------------------

```
$ berks
...
Fetching 'redisio' from https://github.com/leaprail/redisio.git (at chef-13)
Git error: command `git clone https://github.com/leaprail/redisio.git "/Users/tap/.berkshelf/.cache/git/bb551905ffaed7b4e09723a35c33d6918eb5fa38" --bare --no-hardlinks` failed. If this error persists, try removing the cache directory at '/Users/tap/.berkshelf/.cache/git/bb551905ffaed7b4e09723a35c33d6918eb5fa38'.Output from the command:

Cloning into bare repository '/Users/tap/.berkshelf/.cache/git/bb551905ffaed7b4e09723a35c33d6918eb5fa38'...
fatal: could not read Username for 'https://github.com': Device not configured
```

**solution**

specified repo (`leaprail/redisio`) no longer exists.

Could not satisfy version constraints for: ruby_rbenv
-----------------------------------------------------

```
$ knife zero converge 'name:sith'
...
resolving cookbooks for run list: ["app_sith"]

================================================================================
Error Resolving Cookbooks for Run List:
================================================================================

Missing Cookbooks:
------------------
Could not satisfy version constraints for: ruby_rbenv
```

still I cannot find any version constraints for specified cookbook.

**solution**

try to remove `Berksfile.lock` and run `berks` or `berks vendor` again.
if that doesn't help, remove vendored cookbook from _berks-cookbooks/_
(see [Chef - Tips]({% post_url 2018-01-16-chef-tips %}) on how to update
cookbook).

git-core is a virtual package provided by multiple packages, you must explicitly select one
-------------------------------------------------------------------------------------------

```
$ knife zero converge 'name:sith'
...
Recipe: ruby_rbenv::user_install
  * apt_package[git-core] action install

    ================================================================================
    Error executing action `install` on resource 'apt_package[git-core]'
    ================================================================================

    Chef::Exceptions::Package
    -------------------------
    git-core is a virtual package provided by multiple packages, you must explicitly select one

    Resource Declaration:
    ---------------------
    # In /var/chef/cache/cookbooks/ruby_rbenv/libraries/chef_rbenv_recipe_helpers.rb

     34:         node['rbenv']['install_pkgs'].each { |pkg| package pkg }
     35:       end
```

**solution**

server OS: Ubuntu 18.04 LTS.

_berks-cookbooks/ruby_rbenv/attributes/default.rb_:

```ruby
case node['platform_family']
# ...
when 'debian', 'suse'
  default['rbenv']['install_pkgs'] = %w(git-core grep)
  default['rbenv']['user_home_root'] = '/home'
```

solution is to use `git` package instead of `git-core`. it's possible to
override this attribute in application cookbook until it's fixed in repo:

```ruby
override['rbenv']['install_pkgs'] = %w[git-core grep]
```

No candidate version available for libgdbm3
-------------------------------------------

server OS: Ubuntu 18.04 LTS.

```
$ knife zero converge 'name:sith'
...
Recipe: ruby_rbenv::user
  ...
  * apt_package[libgdbm3] action install
    * No candidate version available for libgdbm3
    ================================================================================
    Error executing action `install` on resource 'apt_package[libgdbm3]'
    ================================================================================

    Chef::Exceptions::Package
    -------------------------
    No candidate version available for libgdbm3
```

**solution**

`ruby-rbenv` tries to install outdated package - `libgdbm4` package
should be installed instead of `libgdbm3` one.

solution is to update `ruby-rbenv` cookbook that has correct dependencies.
