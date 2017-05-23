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

### network

[man interfaces 5](https://manpages.debian.org/jessie/ifupdown/interfaces.5.en.html)

- list network interfaces with their statuses and MAC addresses

  ```sh
  $ ip link
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
  2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
      link/ether b8:XX:XX:XX:XX:6d brd ff:ff:ff:ff:ff:ff
  3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DORMANT group default qlen 1000
      link/ether b8:XX:XX:XX:XX:38 brd ff:ff:ff:ff:ff:ff
  ```

- list network interfaces (like `ip link` but with IP addresses)

  ```sh
  $ ip address
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
        valid_lft forever preferred_lft forever
  2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether b8:XX:XX:XX:XX:6d brd ff:ff:ff:ff:ff:ff
      inet 192.168.0.131/24 brd 192.168.0.255 scope global eth0
        valid_lft forever preferred_lft forever
      inet6 fe80::3c44:72e7:62ef:445/64 scope link
        valid_lft forever preferred_lft forever
  3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether b8:XX:XX:XX:XX:38 brd ff:ff:ff:ff:ff:ff
      inet 192.168.0.130/24 brd 192.168.0.255 scope global wlan0
        valid_lft forever preferred_lft forever
      inet6 fe80::ba27:ebff:fef5:1138/64 scope link
        valid_lft forever preferred_lft forever
  ```

- bring interface up/down (using `ifupdown` package)

  ```sh
  $ sudo ifup eth0
  $ sudo ifdown eth0
  ```

- restart `networking` service

  ```sh
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart networking
  ```

- disable interface (say, `wlan0`)

  _/etc/network/interfaces_:

  ```sh
  # or else wlan0 will be up after reboot
  #allow-hotplug wlan0
  # or else wlan0 will be up after reboot
  # or restarting networking service
  #auto wlan0
  # set method to manual instead of dhcp or static
  iface wlan0 inet manual
  ```

  NOTE: if you set method to manual for `eth0` interface it's still up
        after rebooting RPI even though appears to be not configured
        (trying to bring it down with `ifdown` command returns error
        unless you bring it up explicitly with `ifup` command).

- apply new network settings

  restart interface (say, `eth0`):

  ```sh
  $ sudo ifdown eth0
  $ sudo ifup eth0
  ```

  or restart `networking` service:

  ```sh
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart networking
  ```

  or just reboot RPI:

  ```sh
  $ sudo systemctl reboot
  ```

#### Ethernet

<https://www.mathworks.com/help/supportpkg/raspberrypi/ug/getting-the-raspberry_pi-ip-address.html>

make sure RPI Ethernet lights are on during boot - I used a cable with 4P4C
(instead of 8P8C) connectors by mistake and my router refused to talk to RPI.

- reserve IP address for Ethernet adapter in your router
  (it has different MAC address than Wi-Fi adapter)
- connect Ethernet cable and boot RPI
- configure either `eth0` interface to use either static or dhcp method

  _/etc/network/interfaces_ (static method):

  ```sh
  iface eth0 inet static
      address 192.168.0.131
      netmask 255.255.255.0
      gateway 192.168.0.1
  ```

  _/etc/network/interfaces_ (dhcp method):

  ```sh
  iface eth0 inet dhcp
  ```

- apply new network settings (see instructions in `network` section)

#### Wi-Fi

<https://wiki.debian.org/WiFi/HowToUse>

NOTE: network module in RPI supports 2.4 GHz only
      (at least `iwlist scan` doesn't show 5 GHz networks).

NOTE: pay attention to using double quotes around PSK (pre-shared key) in
      configuration files - only plain passphrase must be quoted, PSK is
      typed directly without quotes.

- generate wpa psk for your Wi-Fi network

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

- apply new network settings (see instructions in `network` section)

##### using WPA supplicant

<https://wiki.archlinux.org/index.php/WPA_supplicant>

- create configuration file

  default configuration file is _/etc/wpa_supplicant/wpa_supplicant.conf_.

  ```sh
  $ sudo cp wpa_supplicant.conf wlan0.conf
  ```

  _/etc/wpa_supplicant/wlan0.conf_:

  ```sh
  country=RU
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1
  network={
          ssid="<myssid>"
          psk=<mypsk>
  }
  ```

  the difference from default configuration file:

  - country is modified
  - network section with credentials is added

  network section above can be generated using `wpa_passphrase` command:

  ```sh
  $ wpa_passphrase myssid mypassphrase
  network={
          ssid="myssid"
          #psk="mypassphrase"
          psk=af3492c3f8040dd43589d2700bbeacc7d6aa60e91f2225fe29898769fa139965
  }
  ```

  multiple network sections are allowed:

  ```sh
  country=RU
  ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
  update_config=1

  network={
          ssid="<myssid_1>"
          psk=<mypsk_1>
  }
  network={
          ssid="<myssid_2>"
          psk="<mypassphrase_2>"
  }
  ```

  NOTE: if using passphrase its length must be at least 8 characters -
        otherwise trying to bring `wlan0` interface up will fail with
        not very informative message:

  ```sh
  $ sudo ifup wlan0
  wpa_supplicant: /sbin/wpa_supplicant daemon failed to start
  run-parts: /etc/network/if-pre-up.d/wpasupplicant exited with return code 1
  Failed to bring up wlan0.
  ```

- use new configuration file for `wlan0` interface

  _/etc/network/interfaces_:

  ```sh
  allow-hotplug wlan0
  auto wlan0
  iface wlan0 inet dhcp
      wpa-conf /etc/wpa_supplicant/wlan0.conf
  ```

- apply new network settings (see instructions in `network` section)

### packages

```sh
$ sudo apt update
$ sudo apt install locate mc htop omxplayer vim git iotop
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
build it from source or install package from stretch release instead.

install global gems after installing Ruby:

```sh
$ sudo gem install bundler git-up
```

#### build from source

install packages for openssl and readline extensions:

```sh
$ sudo apt install libssl-dev libreadline-dev
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

#### install package from stretch release

<https://raspberrypi.stackexchange.com/questions/49567#49567>

_/etc/apt/sources.list_:

```
deb http://archive.raspbian.org/raspbian/ stretch main
```

```sh
$ sudo apt update
$ sudo apt install ruby2.3
```

it's necessary to remove Ruby built from source in order to use Ruby installed
from repository (unless you use Ruby version managers such as RVM or rbenv):

```sh
$ cd ~/tmp/ruby-2.4.1
$ sudo make uninstall
```

`uninstall` target is available only if you installed Ruby previously from the
same directory - otherwise run `./configure && make && sudo make install` again.

also after downgrading Ruby version it might be necessary to update RubyGems
(package manager for Ruby which provides `gem` command):

```sh
$ gem update --system
```

or else you might encounter this error when trying to install global gems:

```sh
$ sudo gem install bundler git-up
ERROR:  While executing gem ... (TypeError)
    no implicit conversion of nil into String
```

### SSH

- add RPI host record to _~/.ssh/config_ on local machine
- login to RPI with login/password first
- add your public RSA key to _~/.ssh/authorized_keys_ on RPI

### Samba

<https://wiki.debian.org/SambaServerSimple>

- install Samba server and client

  ```sh
  $ sudo apt install samba samba-client
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
  $ sudo apt remove <packagename>
  ```

  remove everything regarding package but without dependencies:

  ```sh
  $ sudo apt [purge|remove --purge] <packagename>
  ```

  remove all orphaned packages:

  ```sh
  $ sudo apt autoremove
  ```

  remove everything regarding package with dependencies:

  ```sh
  $ sudo aptitude [remove|purge] <packagename>
  ```
