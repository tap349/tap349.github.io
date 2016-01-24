---
layout: post
title: actions to converge umka with chef
date: 2016-01-24 20:42:00 +0300
access: public
categories: [chef]
---

on local workstation:

```bash
$ brew cask update
$ brew cask install chefdk
```

on remote node:

```bash
# useradd deploy -m -p deploy -G sudo
# su - deploy
# groups deploy
```

on local workstation:

```bash
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

