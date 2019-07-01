---
layout: post
title: Zsh startup files
date: 2017-03-07 12:58:19 +0300
access: public
comments: true
categories: [zsh]
---

loading order of Zsh startup files:

_$ZDOTDIR/.zshenv_ → _$ZDOTDIR/.zshrc_ → _$ZDOTDIR/.zlogin_

- interactive login shell (`zsh -il -c "cmd"` or opening new tab in iTerm2):

  - _$ZDOTDIR/.zshenv_
  - _$ZDOTDIR/.zshrc_
  - _$ZDOTDIR/.zlogin_

- interactive shell (`zsh -i -c "cmd"`):

  - _$ZDOTDIR/.zshenv_
  - _$ZDOTDIR/.zshrc_

- login shell (`zsh -l -c "cmd"`):

  - _$ZDOTDIR/.zshenv_
  - _$ZDOTDIR/.zlogin_

- non-interactive and not login shell (`zsh -c "cmd"`):

  - _$ZDOTDIR/.zshenv_
