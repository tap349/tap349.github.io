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

- install all system updates in App Store
- install brew (<http://brew.sh/index.html>):

  ```sh
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

  NOTE: it will also download and install CLT for Xcode.

- `brew install gpg git` (for decrypting ssh archive and cloning git repos)
- copy ssh keys into _~/.ssh_:
  - `gpg -d --output ssh.tar.gz ssh.tar.gz.gpg`
  - `tar xvzf ssh.tar.gz`
- `git clone git@github.com:tap349/dotfiles.git ~/.dotfiles`
- `git clone git@github.com:tap349/tap349.github.io.git ~/blog`
- `cd ~/.dotfiles && brew bundle -v` and postinstallation setup
  (see comments in _Brewfile_)
- `~/.dotfiles/make_symlinks.sh`
- Battery: Show percentage
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
  - [x] Minimize windows into application icon
  - [x] Automatically hide and show the Dock

### Security & Privacy

  - General:
    - [x] Require password 5 seconds after sleep or screen saver begins

### Displays

  - [ ] Show mirroring options in the menu bar when available

### Keyboard

  - Keyboard:
    - Key Repeat: Fast (max)
    - Delay Until Repeat: Short (max)
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
    - [x] Look up & data detectors
    - [x] Secondary click: Click or tap with two fingers
    - [x] Tap to click
  - More gestures:
    - [x] App Expose: Swipe down with three fingers

### Sharing

  - Computer Name: MacBook Pro Personal
