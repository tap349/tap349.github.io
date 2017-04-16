---
layout: post
title: raspberry pi
date: 2017-04-15 22:46:59 +0300
access: public
categories: [pi]
---

<!-- more -->

* TOC
{:toc}

## installation

<https://www.raspberrypi.org/downloads/raspbian/>

- download `raspbian jessie lite` image
- restore image to microsd with `applepi-baker`
- create empty _ssh_ file in microsd root to enable ssh
  (do it before the very first boot!)

## configuration

### keyboard

<https://wiki.debian.org/Keyboard>

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

```sh
$ sudo dpkg-reconfigure locales
$ logout
```

- Locales to be generated:
  - [x] `en_US.UTF-8 UTF-8`
- Default locale for the system environment: `en_US.UTF-8`

when connecting via ssh remote host will try to set the same
locale as on local computer - this operation will fail if
corresponding locale is not generated on remote server:

> -bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)

that is why it's a good idea to have the same locale on both
local computer and remote host.

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
```

### timezone

```sh
$ sudo timedatectl set-timezone Europe/Moscow
$ date
Sun 16 Apr 00:55:55 MSK 2017
```

### wi-fi

<https://wiki.debian.org/WiFi/HowToUse>

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
$ sudo apt-get install mc htop omxplayer
```

### screen resolution

- list all modes for `DMT` hdmi group

  ```sh
  $ tvservice -m DMT
  ```

  current mode is marked as `(prefer)` (`mode 58: 1680x1050` by default).
  I need `mode 82: 1920x1050`.

- set framebuffer width/height and hdmi mode

  _/boot/config.txt_:

  ```config
  framebuffer_width=1920
  framebuffer_height=1080

  hdmi_group=2 # CEA - 1, DMT - 2
  hdmi_mode=82
  ```

- reboot

### ruby

install packages for openssl and readline extensions:

```sh
$ sudo apt-get install libssl-dev libreadline-dev
```

run `make clean` if you have already tried to compile ruby
before and got warnings about not installed extensions
(because of not installed packages mentioned above).

```sh
$ cd ~/tmp
$ wget http://cache.ruby-lang.org/pub/ruby/2.4/ruby-2.4.1.tar.gz
$ tar xvzf ruby-2.4.1.tar.gz
$ cd ruby-2.4.1
$ ./configure && make && sudo make install
$ cd .. && rm -rf ruby-2.4.1.tar.gz ruby-2.4.1/
```
