---
layout: post
title: actions to converge umka with chef
date: 2016-01-24 20:42:00 +0300
access: public
categories: [chef]
---

list of commands required to prepare and cook umka node

<!-- more -->

- http://www.creationline.com/en/lab/8323
>knife-zero starts chef-zero server locally (it's used to store node policies),
>logs into remote node via ssh and runs chef-client there using policies from
>local chef-zero server.

- https://www.coveros.com/knife-zero/
>This plugin creates an ssh tunnel so that
>when chef-zero listens on the master node,
>and the remote node tries to http connect to itself,
>it actually tunnels back to the listening master node.

the point why remote node "tries to http connect to itself":

_/etc/chef/client.rb_:

```ruby
chef_server_url  "chefzero://localhost:8889"
```

## remote node

NOTE: `tap349` is a preconfigured host in _~/.ssh/config_ with root user
      and local private key added to authorized keys on remote node.

- create `deploy` user and add him to `sudo` group:

    ```sh
    # ssh tap349
    # useradd deploy -m -G sudo
    # passwd deploy
      Enter new UNIX password: deploy
      Retype new UNIX password: deploy
      passwd: password updated successfully
    # su - deploy
    $ groups
      deploy sudo
    ```

## local workstation

- install chefdk:

    ```sh
    $ brew cask update
    $ brew cask install chefdk
    ```


- update chef inside chef-dk since knife-zero needs chef ~> 12.6.0:

  - https://github.com/chef/appbundle-updater
  - https://rvm.io/integration/sudo

    ```sh
    $ gem install appbundle-updater
    $ rvmsudo appbundle-updater chefdk chef master
    $ chef --version
      Chef Development Kit Version: 0.10.0
      chef-client version: 12.6.0
      berks version: 4.0.1
      kitchen version: 1.4.2
    ```

- install knife-zero chef plugin:

  - http://www.creationline.com/en/lab/8323
  - https://github.com/higanworks/knife-zero
  - https://www.coveros.com/knife-zero/

    ```sh
    $ chef gem install knife-zero
    ```

- generate chef repo:

    ```sh
    $ chef generate repo umka
    $ tree -aF -L 3 -I .git umka

    umka
    ├── .chef-repo.txt
    ├── .gitignore
    ├── LICENSE
    ├── README.md
    ├── chefignore
    ├── cookbooks/
    │   ├── README.md
    │   └── example/
    │       ├── README.md
    │       ├── attributes/
    │       ├── metadata.rb
    │       └── recipes/
    ├── data_bags/
    │   ├── README.md
    │   └── example/
    │       └── example_item.json
    ├── environments/
    │   ├── README.md
    │   └── example.json
    └── roles/
        ├── README.md
        └── example.json
    ```

- add new host to ssh config:

    _~/.ssh/config_:

    ```
    Host tap349-deploy
    User deploy
    Hostname DIGITAL_OCEAN_IP
    IdentityFile ~/.ssh/home/id_rsa
    ForwardAgent yes
    ServerAliveInterval 180
    ```

- add your public key to authorized keys on remote node:

    ```sh
    $ ssh deploy@tap349-deploy 'mkdir -p ~/.ssh'
      deploy@DIGITAL_OCEAN_IP's password: deploy
    $ cat ~/.ssh/home/id_rsa.pub | ssh deploy@tap349-deploy "cat >> ~/.ssh/authorized_keys"
      deploy@DIGITAL_OCEAN_IP's password: deploy
    ```

    now login with `ssh tap349-deploy` should work without password.

- bootstrap remote node:

    ```sh
    $ knife zero bootstrap tap349-deploy --sudo
      WARNING: No knife configuration file found
      Doing old-style registration with the validation key at ...
      Delete your validation key in order to use your user credentials instead

      Connecting to tap349-deploy
      tap349-deploy knife sudo password:
      Enter your password:
      tap349-deploy
      tap349-deploy -----> Installing Chef Omnibus (-v 12)
      tap349-deploy downloading https://omnitruck-direct.chef.io/chef/install.sh
      tap349-deploy   to file /tmp/install.sh.4084/install.sh
      tap349-deploy trying wget...
      tap349-deploy Getting information for chef stable 12 for ubuntu...
      tap349-deploy downloading https://omnitruck-direct.chef.io/stable/chef/metadata?v=12&p=ubuntu&pv=14.04&m=x86_64
      tap349-deploy   to file /tmp/install.sh.4088/metadata.txt
      tap349-deploy trying wget...
      tap349-deploy url	https://opscode-omnibus-packages.s3.amazonaws.com/ubuntu/14.04/x86_64/chef_12.6.0-1_amd64.deb
      tap349-deploy md5	5cfc19d5a036b3f7860716bc9795a85e
      tap349-deploy sha256	e0b42748daf55b5dab815a8ace1de06385db98e29a27ca916cb44f375ef65453
      tap349-deploy version	12.6.0downloaded metadata file looks valid...
      tap349-deploy downloading https://opscode-omnibus-packages.s3.amazonaws.com/ubuntu/14.04/x86_64/chef_12.6.0-1_amd64.deb
      tap349-deploy   to file /tmp/install.sh.4088/chef_12.6.0-1_amd64.deb
      tap349-deploy trying wget...
      tap349-deploy Comparing checksum with sha256sum...
      tap349-deploy Installing chef 12
      tap349-deploy installing with dpkg...
      tap349-deploy Selecting previously unselected package chef.
      (Reading database ... 146472 files and directories currently installed.)
      tap349-deploy Preparing to unpack .../chef_12.6.0-1_amd64.deb ...
      tap349-deploy Unpacking chef (12.6.0-1) ...
      tap349-deploy Setting up chef (12.6.0-1) ...
      tap349-deploy Thank you for installing Chef!
      tap349-deploy Starting the first Chef Client run...
      tap349-deploy Starting Chef Client, version 12.6.0
      tap349-deploy Creating a new client identity for tap349 using the validator key.
      tap349-deploy resolving cookbooks for run list: []
      tap349-deploy Synchronizing Cookbooks:
      tap349-deploy Compiling Cookbooks...
      tap349-deploy [2016-01-26T06:31:23-05:00] WARN: Node tap349 has an empty run list.
      tap349-deploy Converging 0 resources
      tap349-deploy
      tap349-deploy Running handlers:
      tap349-deploy Running handlers complete
      tap349-deploy Chef Client finished, 0/0 resources updated in 03 seconds
    ```
