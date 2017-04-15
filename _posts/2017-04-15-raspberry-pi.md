---
layout: post
title: raspberry pi
date: 2017-04-15 22:46:59 +0300
access: public
categories: [pi]
---

<!-- more -->

## installation

- download `raspbian jessie lite` image
- restore image to microsd with `applepi-baker`
- create empty _ssh_ file in microsd root to enable ssh
  (do it before the very first boot!)

## configuration

### dvorak

<https://wiki.debian.org/Keyboard>

```sh
$ sudo dpkg-reconfigure keyboard-configuration
$ sudo service keyboard-setup restart
```

or else edit _/etc/default/keyboard_ manually:

```config
XKBMODEL="macintosh_hhk"
XKBLAYOUT="us"
XKBVARIANT="dvorak"
XKBOPTIONS=""

BACKSPACE="guess"
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
  # ping ya.ru
  ```

### packages

```sh
$ sudo apt-get update
$ sudo apt-get install mc
```

### resolution

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
