---
layout: post
title: iTerm and mc
date: 2016-05-03 11:58:58 +0300
access: public
categories: [iterm, mc]
---

iTerm and mc tips.

<!-- more -->

## use Emacs keybindings (`<M-b>`/`<M-f>`) to navigate between words

*Preferences -> Profiles -> Default -> Keys -> Keyword Behavior*:

`Left option key acts as: +Esc`

## paste from clipboard with right click

*Preferences -> Pointer -> Right button single click*:

`Action: Paste from Clipboard`

## system-wide hotkey to switch to iTerm2

*Preferences -> Keys -> Hotkey*:

- [x] `Show/hide iTerm2 with a system-wide hotkey`
- `Hotkey: <C-D-t>`

## mc is very slow when switching tabs

remove this line from _~/.config/mc/panels.ini_:

```
other_dir=/Users/tap/.dotfiles
```
