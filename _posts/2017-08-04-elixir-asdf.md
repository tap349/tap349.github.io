---
layout: post
title: Elixir - asdf
date: 2017-08-04 12:35:57 +0300
access: public
comments: true
categories: [asdf]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/asdf-vm/asdf>

don't install it via brew because:

- brew formula places asdf files into _/usr/local/opt/asdf/_

  it would be necessary to source _/usr/local/opt/asdf/asdf.sh_ instead of
  _~/.asdf/asdf.sh_ while according to official asdf docs it's required to
  source the latter.

- asdf just doesn't work

  after installing Elixir plugin and setting current version, `elixir`
  executable is not found when asdf is installed via brew.

```sh
$ brew uninstall --force elixir
$ rm -rf ~/.mix/
$ rm -rf ~/.asdf/
$ git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.6.3
$ asdf plugin-add elixir
$ asdf list-all elixir
$ asdf install elixir 1.8.0
$ asdf global elixir 1.8.0
$ asdf current elixir
$ echo 'source $HOME/.asdf/asdf.sh' >> ~/.zshenv
$ elixir -v
$ cd <project>
$ mix compile
```

<https://github.com/asdf-vm/asdf#the-tool-versions-file>:

> Add a .tool-versions file to your project dir and versions of those tools
> will be used. Global defaults can be set in the file $HOME/.tool-versions

`asdf global` command writes specified tool version to _$HOME/.tool-versions_.

all Elixir versions managed by asdf are stored in _~/.asdf/installs/elixir/_.
