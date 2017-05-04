---
layout: post
title: Raspberry Pi
date: 2017-04-15 22:46:59 +0300
access: public
categories: [rpi]
---

<!-- more -->

* TOC
{:toc}

## installation

- <https://www.raspberrypi.org/downloads/raspbian/>
- <https://www.raspberrypi.org/documentation/installation/sdxc_formatting.md>
- <https://www.raspberrypi.org/documentation/installation/installing-images/mac.md>
- <http://www.macworld.co.uk/how-to/mac/how-to-set-up-raspberry-pi-3-with-mac-3637490/>

- download `raspbian jessie lite` image
- use `applepi-baker` to write it to microsd
  (or else use command line - see link above for details)
- create empty _ssh_ file in microsd root to enable SSH
  (do it before the very first boot!)

## configuration

### keyboard

<https://wiki.debian.org/Keyboard>

#### dpkg-reconfigure

```sh
$ sudo dpkg-reconfigure keyboard-configuration
$ sudo service keyboard-setup restart
```

- Keyboard model: `Happy Hacking Keyboard for Mac`
- Keyboard layout: `English (US) - English (Dvorak)`
- Key to function as AltGr: `The default for the keyboard layout`
- Compose key: `No compose key`

#### manually

_/etc/default/keyboard_:

```config
XKBMODEL="macintosh_hhk"
XKBLAYOUT="us"
XKBVARIANT="dvorak"
XKBOPTIONS=""

BACKSPACE="guess"
```

```sh
$ sudo service keyboard-setup restart
```

### password

default login/password: `pi`/`raspberry`

```sh
$ passwd
```

### console setup

#### dpkg-reconfigure

```sh
$ sudo dpkg-reconfigure console-setup
```

- Encoding to use on the console: `UTF-8`
- Character set to support: `# Cyrillic - Slavic languages`
- Font for the console: `Terminus`
- Font size: `10x20 (framebuffer only)`

#### manually

_/etc/default/console-setup_:

```config
ACTIVE_CONSOLES="/dev/tty[1-6]"

CHARMAP="UTF-8"

CODESET="CyrSlav"
FONTFACE="Terminus"
FONTSIZE="10x20"

VIDEOMODE=
```

### locale

#### dpkg-reconfigure

```sh
$ sudo dpkg-reconfigure locales
$ logout
/ login
```

- Locales to be generated:
  - [x] `en_US.UTF-8 UTF-8`
- Default locale for the system environment: `en_US.UTF-8`

when connecting via SSH remote host will try to set the same
locale as on local machine - this operation will fail if
corresponding locale is not generated on remote server:

> -bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)

that is why it's a good idea to have the same locale on both
local machine and remote host.

#### manually

_/etc/default/locale_:

```config
LANG=en_US.UTF-8
```

_/etc/locale.gen_:

```config
...
# en_US.ISO-8859-15 ISO-8859-15
en_US.UTF-8 UTF-8
# en_ZA.ISO-8859-1
...
```

```sh
$ sudo locale-gen
$ logout
/ login
```

### timezone

```sh
$ sudo timedatectl set-timezone Europe/Moscow
$ date
Sun 16 Apr 00:55:55 MSK 2017
```

### wi-fi

<https://wiki.debian.org/WiFi/HowToUse>

NOTE: network module in RPI supports 2.4 GHz only
      (at least `iwlist scan` doesn't show 5 GHz networks).

- generate wpa psk for your wi-fi network

  ```sh
  $ sudo su
  # iwlist wlan0 scan | less
  # wpa_passphrase <myssid> <mypassphrase> >> /etc/network/interfaces
  ```

- configure `wlan0` interface

  _/etc/network/interfaces_:

  ```config
  allow-hotplug wlan0
  auto wlan0
  iface wlan0 inet dhcp
      wpa-ssid <myssid>
      wpa-psk <mypsk>

  # don't forget to remove output from wpa_passphrase
  ```

- restart `wlan0` interface

  ```sh
  # ifdown wlan0
  # ifup wlan0
  ```

### packages

```sh
$ sudo apt-get update
$ sudo apt-get install locate mc htop omxplayer vim git iotop
```

#### vim

_~./vimrc_:

```vim
set showmode

set hlsearch
set incsearch
```

### bash

_~/.bash_aliases_:

```sh
alias g='git'
alias ga='git add -A .'
alias gd='git diff'
alias gdc='git diff --cached'
alias gs='git status'

alias ll='ls -al'
alias mcu='mc -ub'

alias j='sudo journalctl -ef -u ntv-*'
```

_~/.bashrc_:

```sh
...
export EDITOR=vim
```

### screen resolution

#### resolution

NOTE: HDMI cable must be plugged in before booting RPI -
      otherwise HDMI signal is not detected.

- list all modes for `DMT` HDMI group

  ```sh
  $ tvservice -m DMT
  ```

  current mode is marked as `(prefer)` (`mode 58: 1680x1050` by default).
  I need `mode 82: 1920x1050`.

- set framebuffer width/height and HDMI mode

  _/boot/config.txt_:

  ```config
  framebuffer_width=1920
  framebuffer_height=1080

  hdmi_group=2 # CEA - 1, DMT - 2
  hdmi_mode=82
  ```

- reboot

#### blanking

<https://raspberrypi.stackexchange.com/a/61080>

- disable screen blanking in console

  add `consoleblank=0` to the end of the line in _/boot/cmdline.txt_.

- reboot

### Ruby

package manager will install an outdated version of Ruby -
build it from source instead.

install packages for openssl and readline extensions:

```sh
$ sudo apt-get install libssl-dev libreadline-dev
```

run `make clean` if you have already tried to compile Ruby
before and got warnings about not installed extensions
(because corresponding dev packages were not installed).

```sh
$ cd ~/tmp
$ wget http://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.1.tar.gz
$ tar xvzf ruby-2.4.1.tar.gz
$ cd ruby-2.4.1
$ ./configure && make && sudo make install
$ cd .. && rm -rf ruby-2.4.1.tar.gz ruby-2.4.1/
```

install global gems:

```sh
$ sudo gem install bundler git-up
```

### SSH

- add RPI host record to _~/.ssh/config_ on local machine
- login to RPI with login/password first
- add your public RSA key to _~/.ssh/authorized_keys_ on RPI

### Samba

<https://wiki.debian.org/SambaServerSimple>

- install Samba server and client

  ```sh
  $ sudo apt-get install samba samba-client
  ```

- configure shared directories

  _/etc/samba/smb.conf_:

  ```config
  # export all home directories
  [homes]
    ...
    # allow to write to them
    read only = no
    ...
    # ensure that only 'username' can connect to //server/username
    valid users = %S
  ```

- add Samba user

  Samba doesn't use system users and has its own password system.

  ```sh
  $ sudo smbpasswd -a pi
  / enter password
  $ sudo pdbedit -wL
  ```

- restart Samba daemon

  ```sh
  $ sudo service smbd restart
  ```

- mount shared directory on local machine

  _~/scripts/mount_pi_:

  ```sh
  mkdir /Volumes/pi
  mount_smbfs //pi@192.168.0.105/pi /Volumes/pi/
  ```

  NOTE: if user is not specified name of currently logged in user is used
        (this user might not exist on Samba server).

  also usually I add new mount points to `Directory hotlist` in mc.

### shutdown

by default `sudo shutdown` schedules shutdown in 1 minute -
run `sudo shutdown now` to shutdown immediately.

<https://www.raspberrypi.org/documentation/installation/sd-cards.md>:

```sh
$ sudo halt
```

### useful commands

- locate any file:

  ```sh
  $ sudo updatedb
  $ sudo locate <filename>
  ```

- list all files of specified package

  ```sh
  $ sudo dpkg-query -L <packagename>
  ```

- search for packages containing specified file

  ```sh
  $ sudo dpkg-query -S <filename>
  ```

- remove package

  remove only binaries:

  ```sh
  $ sudo apt-get remove <packagename>
  ```

  remove everything regarding package but without dependencies:

  ```sh
  $ sudo apt-get [purge|remove --purge] <packagename>
  ```

  remove all orphaned packages:

  ```sh
  $ sudo apt-get autoremove
  ```

  remove everything regarding package with dependencies:

  ```sh
  $ sudo aptitude [remove|purge] <packagename>
  ```
