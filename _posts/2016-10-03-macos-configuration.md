---
layout: post
title: macOS - Configuration
date: 2016-10-03 01:42:54 +0300
access: public
comments: true
categories: [macos]
---

steps to configure new macOS installation.

<!-- more -->

* TOC
{:toc}
<hr>

basic setup
-----------

NOTE: to backup files on external NTFS HDD use Paragon Driver for Mac OS
      (<http://www.seagate.com/support/downloads/item/ntfs-driver-for-mac-os-master-dl>).

- install all system updates in App Store
- install brew:

  ```sh
  $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
  ```

  NOTE: it will also download and install CLT for Xcode.

- `brew install gpg git` (for decrypting SSH archive and cloning git repos)
- copy SSH keys into _~/.ssh_:
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
  - oh-my-zsh installation script will rename existing _~/.zshrc_ into
    _.zshrc.pre-oh-my-zsh_ -\> symlink _~/.zshrc_ into home directory
    after oh-my-zsh is installed

  this script:

  - creates symlinks for all config files and directories
  - copies fonts into _~/Library/Fonts/_

- `mkdir ~/dev` and git clone all repos
- menu bar: `Battery` -\> `Show percentage`
- add PureVPN service:

  <https://support.purevpn.com/how-to-setup-purevpn-l2tp-manually-on-mac>

- application icons in Dock:
  - Finder
  - App Store
  - System Preferences
  - Safari
  - Google Chrome
  - MacVim
  - iTerm2
  - 2Do
  - Telegram Desktop
- Dictionary:
  - select `New Oxford American Dictionary (American English)` and
    `Oxford Russian Dictionary (Russian-English)`
- Notes:
  - Preferences:
    - check `Enable the On My Mac account`
    - set `Default text size` to next to the last
- add second Desktop
- install ruby and gems: [rbenv]({% post_url 2016-03-30-rbenv %})
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

  - TopTracker
  - erlang-history (from github)
  - Tomighty

    Preferences:
      - General:
        - Show time in status bar: Minutes only
      - Time:
        - [ ] Start next timer automatically
      - Notifications:
        - Play tick tock sound during:
          - [ ] Pomodoro
      - Hotkeys:
        - Start Pomodoro: `<C-D-p>` (displayed in qwerty: `<C-D-r>`)
        - Stop Pomodoro: `<C-D-s>` (displayed in qwerty: `<C-D-;>`)

  - GoPanda2

  NOTE: additional software is usually stored in _~/soft/_ to keep it all
        in a single place since it's not managed by any package manager.
- copy other files from backup (to name a few):
  - _~/Pictures/Lightroom/_
  - _~/backup/_
  - _~/docs/_
  - _~/edev/_
  - _~/media/_
  - _~/soft/_
- sign in to iTunes Store

System Preferences
------------------

### Dock

- Position on screen: Right
- [x] Minimize windows into application icon
- [x] Automatically hide and show the Dock
- [ ] Show recent applications in Dock

### Security & Privacy

- General (tab):
  - [x] Require password 5 seconds after sleep or screen saver begins
  - Allow apps downloaded from: App Store and identified developers

### Spotlight

- Search Results (tab):
  - Only selected categories will appear in Spotlight search results:
    - [x] Applications
    - [x] Calculator
    - [x] Conversion
    - [x] Definition
    - [x] Events & Reminders
    - [x] System Preferences
  - [ ] Allow Spotlight Suggestions in Look up
- Privacy (tab):
  - Prevent Spotlight from searching these locations:
    - Downloads
    - media

### Displays

- Turn display off after: 10 min

### Energy Saver

- [ ] Show mirroring options in the menu bar when available

### Keyboard

- Keyboard (tab):
  - Key Repeat: Fast (max)
  - Delay Until Repeat: Short (max)
  - [x] Use all F1, F2, etc. keys as standard function keys
  - Modifier Keys:
    - Caps Lock Key: Control
- Text (tab):
  - [ ] Correct spelling automatically
- Shortcuts (tab):
  - Input Sources (available after adding 2nd input source):
    - [x] Select the previous input source: Command-Space
  - Spotlight:
    - [x] Show Spotlight search: Control-Space
  - App Shortcuts:
    - Google Chrome (see [macOS - Troubleshooting]({% post_url 2017-10-29-macos-troubleshooting %})):
      - Select Previous Tab: `<D-S-[>`
      - Select Next Tab: `<D-S-[>`
- Input Sources (tab):
  - Dvorak
  - Russian - PC

### Mouse

- Point & Click (tab):
  - [x] Secondary click: Click on right side
- More Gestures (tab):
  - [x] Swipe between pages: Scroll left or right with one finger

### Trackpad

- Point & click (tab):
  - [x] Look up & data detectors
  - [x] Secondary click: Click or tap with two fingers
    (default value after checking `Tap to click` option)
  - [x] Tap to click
- More gestures (tab):
  - [x] Mission Control: Swipe up with four fingers
  - [x] App Expose: Swipe down with four fingers

### Sound

- Sound Effects (tab):
  - [ ] Play user interface sound effects
- Input (tab):
  - Input volume: min (disable microphone)

### iCloud

- [x] iCloud Drive:
  - Options (button):
    - [x] iBooks
    - [x] Reminders
    - [x] Weather
    - [x] System Preferences
- [x] Contacts
- [x] Calendars
- [x] Reminders
- [x] Safari
- [x] Notes
- [x] Keychain
- [x] Find My Mac

### App Store

- [ ] Automatically check for updates

### Network

- PureVPN:
  - [x] Show VPN status in menu bar
    (optionally select `Show Time Connected` in menu bar)

### Sharing

- Computer Name: MacBook Pro Personal

### Accessibility

NOTE: currently disabled because of a significant delay
      when scrolling with inertia in iOS emulator.

- Mouse & Trackpad (left menu):
  - Trackpad Options... (button):
    - [ ] Enable dragging: three fingers drag

font configuration
------------------

these fonts should be installed to _~/Library/Fonts/_ directory:

- unpatched `Andale Mono MT Std` and `Andale Mono MT Std Bold` fonts
  (copy manually from dotfiles)

  this font is one of main ones for MacVim.

- custom `Input` font (download from <http://input.fontbureau.com>)

  this font is one of main ones for MacVim.

  current custom configuration on <http://input.fontbureau.com/download>:

  - Regular: `Input Mono Condensed Light`
  - Bold: `Input Mono Condensed Medium`
  - 2nd variant of all configurable characters
  - Line Spacing: 1.6x

  alternatively use `Andale Mono style`.

  NOTE: to get new downloaded font applied close all Vim windows.

- patched regular `MonacoB`, `MonacoB2` or `Inconsolata LGC` font
  (copy one of them manually from dotfiles and patch with `fontpatcher` script
  unless it's already patched)

  this patched is not used directly but solves 2 problems in MacVim:

  - fixes cyrillic characters (they are no longer bold and ugly)
  - provides Powerline symbols for statusline plugin

  NOTE: fonts patched with `fontpatcher` have `-Powerline` suffix.

- patched or unpatched `Inconsolate LGC` font for iTerm2 (install via brew)
