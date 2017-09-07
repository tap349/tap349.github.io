---
layout: post
title: Elixir - asdf
date: 2017-08-04 12:35:57 +0300
access: public
categories: [asdf]
---

<!-- more -->

* TOC
{:toc}
<hr>

don't install it via brew since brew formula places asdf files into
_/usr/local/opt/asdf/_ - this operation requires sudo privileges.

also after installing Elixir plugin and setting current version
`elixir` executable wasn't found - that is asdf just didn't work.

<https://github.com/asdf-vm/asdf>:

```sh
$ brew uninstall --force elixir
$ rm -rf ~/.mix/
$ git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.3.0
$ asdf plugin-add elixir https://github.com/asdf-vm/asdf-elixir
$ asdf install elixir 1.5.1
$ asdf global elixir 1.5.1
$ asdf current elixir
$ elixir -v
$ cd <project>
$ mix compile
```

now elixir versions are stored in _~/.asdf/installs/elixir/_.

_~/.zshenv_:

```zsh
source $HOME/.asdf/asdf.sh
```