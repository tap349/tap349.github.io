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
- install brew:

  ```sh
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

  NOTE: it will also download and install CLT for Xcode.

- `brew install gpg git` (for decrypting ssh archive and cloning git repos)
- copy ssh keys into _~/.ssh_:
  - `gpg -d --output ssh.tar.gz ssh.tar.gz.gpg`
  - `tar xvzf ssh.tar.gz`
  - `rm ssh.tar.gz`
- `git clone git@github.com:tap349/dotfiles.git ~/.dotfiles`
- `git clone git@github.com:tap349/tap349.github.io.git ~/blog`
- `cd ~/.dotfiles && brew bundle -v`

  rerun command if download request for any app from App Store fails
- postinstallation setup (see comments in _Brewfile_)
- `~/.dotfiles/install.sh`

  NOTE: script must be run after completing postinstallation setup!

  e.g. `~/.oh-my-zsh` must be created before running the script:

  - _~/.oh-my-zsh/_ must exist so that script could copy custom theme there
  - oh-my-zsh installation script will rename existing _.zshrc_ config file
    into _.zshrc.pre-oh-my-zsh_ -> symlink _.zshrc_ into home directory after
    oh-my-zsh is installed

  this script:

  - creats symlinks for all config files and directories
  - copies fonts into _~/Library/Fonts/_
- `mkdir ~/dev` and git clone all repos
- Battery: Show percentage
- add PureVPN service:

  <https://support.purevpn.com/how-to-setup-purevpn-l2tp-manually-on-mac>
- application icons in Dock:
  - Finder
  - App Store
  - System Preferences
  - Safari
  - Google Chrome
  - MacVim
  - iTerm
  - Skype
  - 2Do
- Dictionary:
  - Select `New Oxford American Dictionary (American English)` only
- add second Desktop
- install ruby and gems: [rbenv]({{ site.url }}{% post_url 2016-03-30-rbenv %})
- copy history files from backup:
  - _.pry_history_
  - _.psql_history_
  - _.zsh_history_
- install additional software (must be downloaded manually):
  - Paragon Driver for Mac OS (see above)
  - Adobe Lightroom 5 (latest version as of now - 5.7.1)

    <https://forums.adobe.com/thread/1639590>

    NOTE: app might fail to launch if you opted for case-sensitive filesystem
          when installing macOS. rename offensive directories to lower case
          (see report details for specific path that failed) and don't forget
          to update symlinks.
- copy other files from backup (to name a few):
  - _~/Pictures/Lightroom/_
  - _~/backup/_
  - _~/docs/_
  - _~/edev/_
  - _~/media/_
  - _~/soft/_

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
      - Caps Lock Key: Control
  - Text:
    - [ ] Correct spelling automatically
  - Shortcuts:
    - Input Sources (available after adding 2nd input source):
      - [x] Select the previous input source: Command-Space
    - Spotlight:
      - [x] Show Spotlight search: Control-Space
  - Input Sources:
    - Dvorak
    - Russian - PC

### Mouse

  - Point & Click:
    - [x] Secondary click: Click on right side
  - More Gestures:
    - [x] Swipe between pages: Scroll left or right with one finger

### Trackpad

  - Point & click:
    - [x] Look up & data detectors
    - [x] Secondary click: Click or tap with two fingers
      (default value after checking `Tap to click` option)
    - [x] Tap to click
  - More gestures:
    - [x] App Expose: Swipe down with three fingers

### Sound

  - Sound Effects:
    - [ ] Play user interface sound effects
  - Input:
    - Input volume: min (disable microphone)

### Sharing

  - Computer Name: MacBook Pro Personal

## Troubleshooting

### battery percentage is not properly updating

reset the SMC:

<https://support.apple.com/en-us/HT201295>
