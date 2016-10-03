---
layout: post
title: macOS configuration
date: 2016-10-03 01:42:54 +0300
access: public
categories: [macos]
---

steps to configure new macOS installation.

<!-- more -->

NOTE: to backup files on external NTFS HDD use Paragon Driver for Mac OS
      (<http://www.seagate.com/support/downloads/item/ntfs-driver-for-mac-os-master-dl/>).

- install all updates in App Store
- install brew (<http://brew.sh/index.html>):

  ```sh
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

- `brew install gpg git` (for decrypting ssh archive and cloning git repos)
- copy ssh keys into _~/.ssh_
- `git clone git@github.com:tap349/dotfiles.git ~/.dotfiles`
- `git clone git@github.com:tap349/tap349.github.io.git ~/blog`
- `cd ~/.dotfiles && brew bundle` and postinstallation setup
  (see comments in _Brewfile_)
- `~/.dotfiles/make_symlinks.sh`
- PureVPN (<https://support.purevpn.com/how-to-setup-purevpn-l2tp-manually-on-mac>)
- application icons in Dock:
  - App Store
  - System Preferences
  - Safari
  - Google Chrome
  - MacVim
  - Skype
  - iTerm
  - 2Do

## System Preferences

### Dock

  - Position on screen: Right

### Keyboard

  - Keyboard:
    - [x] Use all F1, F2, etc. keys as standard function keys
    - Modifier Keys:
      - Caps Lock Key: <C>
  - Text:
    - [ ] Correct spelling automatically
  - Shortcuts:
    - Input Sources:
      - [x] Select the previous input source: <D-Space>
    - Spotlight:
      - [x] Show Spotlight search: <C-Space>
  - Shortcuts:
    - Dvorak
    - Russian - PC

### Trackpad

  - Point and click:
    - [x] Secondary click: Click or tap with two fingers
    - [x] Tap to click

### Sharing

  - Computer Name: MacBook Pro Personal
