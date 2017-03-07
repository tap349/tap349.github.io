---
layout: post
title: zsh shells
date: 2017-03-07 12:58:19 +0300
access: public
categories: [zsh]
---

order of loading configuration zsh files:

| _~/.zshenv_ -> _~/.zshrc_ -> _~/.zlogin_

interactive login shell (`zsh -il -c "cmd"` or opening new tab in iTerm2):

- _~/.zshenv_
- _~/.zshrc_
- _~/.zlogin_

interactive shell (`zsh -i -c "cmd"`):

- _~/.zshenv_
- _~/.zshrc_

login shell (`zsh -l -c "cmd"`):

- _~/.zshenv_
- _~/.zlogin_

non-interactive and not login shell (`zsh -c "cmd"`):

- _~/.zshenv_
