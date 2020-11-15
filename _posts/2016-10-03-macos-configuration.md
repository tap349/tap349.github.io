---
layout: post
title: macOS - Configuration
date: 2016-10-03 01:42:54 +0300
access: public
comments: true
categories: [macos]
---

<!-- @format -->

steps to configure new macOS installation.

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## basic setup

NOTE: to backup files on external NTFS HDD use Paragon Driver for Mac OS
(<http://www.seagate.com/support/downloads/item/ntfs-driver-for-mac-os-master-dl>).

- install all system updates in the App Store
- install brew:

  ```zsh
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

  rerun command if download request for any app from the App Store fails

- postinstallation setup (see comments in _Brewfile_)
- `~/.dotfiles/install.sh`

  NOTE: script must be run after completing postinstallation setup!

  this script:

  - creates symlinks for all config files and directories
  - copies fonts into _~/Library/Fonts/_

- `mkdir ~/dev` and git clone all repos
- Menu Bar: `Battery` → `Show percentage`
- add PureVPN service:

  <https://support.purevpn.com/how-to-setup-purevpn-l2tp-manually-on-mac>

- application icons in Dock:
  - Finder
  - App Store
  - System Preferences
  - Safari
  - Google Chrome
  - Trello
  - MacVim
  - Telegram Desktop
  - iTerm2
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

  - Adobe Lightroom 5 (the latest version as of now - 5.7.1)

    <https://forums.adobe.com/thread/1639590>

    NOTE: app might fail to launch if you opted for case-sensitive filesystem
    when installing macOS. rename offensive directories to lower case (see
    report details for specific path that failed) and don't forget to update
    symlinks.

  - FonePaw Screen Recorder
  - GoPanda2
  - Grammarly
  - Notion
  - Obsidian
  - Paragon Driver for Mac OS (see above)
  - Tomighty
    - Preferences:
      - General (tab):
        - Show time in status bar: Minutes only
      - Time (tab):
        - [ ] Start next timer automatically
      - Notifications (tab):
        - Play tick tock sound during:
          - [ ] Pomodoro
      - Hotkeys (tab):
        - Start Pomodoro: `<C-D-p>` (displayed in qwerty: `<C-D-r>`)
        - Stop Pomodoro: `<C-D-s>` (displayed in qwerty: `<C-D-;>`)
  - TopTracker

  NOTE: additional software is usually stored in _~/soft/_ to keep it all in a
  single place since it's not managed by any package manager.

- copy other files from backup (to name a few):
  - _~/Pictures/Lightroom/_
  - _~/backup/_
  - _~/docs/_
  - _~/edev/_
  - _~/media/_
  - _~/soft/_
- sign in to iTunes Store

## System Preferences

### Apple ID

- [x] iCloud Drive:
  - Options (button):
    - [x] Books
    - [x] Reminders
    - [x] Weather
    - [x] Voice Memos
    - [x] System Preferences
- [x] Photos
- [ ] Mail
- [x] Contacts
- [x] Calendars
- [x] Reminders
- [x] Safari
- [x] Notes
- [ ] Siri
- [x] Keychain
- [x] Find My Mac
- [ ] News
- [ ] Stocks
- [ ] Home

### General

- [ ] Automatically hide and show the menu bar
- Default web browser: Google Chrome

### Dock

- Position on screen: Right
- [x] Minimize windows into application icon
- [x] Automatically hide and show the Dock
- [ ] Show recent applications in Dock

### Users & Groups

- Login Items (tab):
  - Sip (Hide [ ])
  - Flexiglass (Hide [ ])
  - Cloud Mail.Ru (Hide [ ])
  - smcFanControl (Hide [ ])
  - Flux (Hide [ ])
  - Docker (Hide [ ])

### Accessibility

NOTE: this option might cause significant delay when scrolling with inertia in
iOS emulator.

- Pointer Control (left menu):
  - Mouse & Trackpad (tab):
    - Trackpad Options... (button):
      - [x] Enable dragging: three fingers drag

### Security & Privacy

- General (tab):
  - [x] Require password 5 seconds after sleep or screen saver begins
  - Allow apps downloaded from: App Store and identified developers
- Privacy (tab):
  - [ ] Siri & Dictation
  - [ ] Maps

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
  - [x] Secondary click: Click or tap with two fingers (default value after
        checking `Tap to click` option)
  - [x] Tap to click
- More gestures (tab):
  - [x] Mission Control: Swipe up with four fingers
  - [x] App Expose: Swipe down with four fingers

### Sound

- Sound Effects (tab):
  - [ ] Play user interface sound effects
- Input (tab):
  - Input volume: min (disable microphone)

### Keyboard

- Text (tab):
  - [ ] Use smart quotes and dashes

### Software Update

- [ ] Automatically keep my Mac up to date

### Network

- PureVPN:
  - [x] Show VPN status in menu bar (optionally select `Show Time Connected` in
        menu bar)

### Sharing

- Computer Name: MacBook Pro Personal

### Date & Time

- Clock (tab):
  - [x] Show date and time in menu bar:
    - Time options:
      - [x] Digital
      - [ ] Use a 24-hour clock
      - [x] Show AM/PM
    - Date options:
      - [ ] Show the day of the week
      - [x] Show date

## font configuration

these fonts should be installed to _~/Library/Fonts/_ directory:

- unpatched `Andale Mono MT Std` and `Andale Mono MT Std Bold` fonts (copy
  manually from dotfiles)

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

- patched regular `MonacoB`, `MonacoB2` or `Inconsolata LGC` font (copy one of
  them manually from dotfiles and patch with `fontpatcher` script unless it's
  already patched)

  this patched is not used directly but solves 2 problems in MacVim:

  - fixes cyrillic characters (they are no longer bold and ugly)
  - provides Powerline symbols for statusline plugin

  NOTE: fonts patched with `fontpatcher` have `-Powerline` suffix.

- patched or unpatched `Inconsolate LGC` font for iTerm2 (install via brew)

## MacVim configuration

1. [MacVim]({% post_url 2019-06-21-macvim %})

```zsh
# $ZDOTDIR/.zlogin

defaults write org.vim.MacVim MMShowAddTabButton 0
defaults write org.vim.MacVim MMNoTitleBarWindow 1
defaults write org.vim.MacVim MMZoomBoth 1
defaults write org.vim.MacVim MMTextInsetTop 1
defaults write org.vim.MacVim MMTextInsetRight 3
defaults write org.vim.MacVim MMTextInsetBottom 5
defaults write org.vim.MacVim MMTextInsetLeft 5
defaults write org.vim.MacVim MMFullScreenFadeTime 0
# https://github.com/macvim-dev/macvim/issues/390#issuecomment-254252969
defaults write org.vim.MacVim SUEnableAutomaticChecks 0

# currently unused settings

# https://github.com/macvim-dev/macvim/wiki/FAQ#black-screen-on-full-screen
# it's longer required on macOS Mojave and MacVim 8.1-151_2
#defaults write org.vim.MacVim MMUseCGLayerAlways 1
#defaults write org.vim.MacVim MMNoFontSubstitution 1
#defaults write org.vim.MacVim MMNativeFullScreen 1
#defaults write org.vim.MacVim MMTabMinWidth 120
#defaults write org.vim.MacVim MMTabMaxWidth 250
#defaults write org.vim.MacVim MMTabOptimumWidth 200
```

in fact it's not necessary to put these commands into _\$ZDOTDIR/.zlogin_ and
run them every time new Zsh shell is started (which increases shell startup
time): changes are persisted on OS level (in user defaults database) so it's
enough to run them only once.
