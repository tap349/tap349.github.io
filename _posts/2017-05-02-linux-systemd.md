---
layout: post
title: Linux - systemd
date: 2017-05-02 02:41:19 +0300
access: public
comments: true
categories: [linux, systemd]
---

<!-- more -->

* TOC
{:toc}
<hr>

useful commands
---------------

### systemctl

- reload all unit files

  do it every time after adding or modifying unit files:

  ```sh
  $ sudo systemctl daemon-reload
  ```

- show unit status (+ unit details and recent log entries)

  ```sh
  $ sudo systemctl status my-player.socket
  ```

- list active units by type

  ```sh
  $ sudo systemctl list-units --type socket
  ```

- list all or specified units and show their state (enabled/disabled)

  ```sh
  $ sudo systemctl list-unit-files
  $ sudo systemctl list-unit-files --type socket
  $ sudo systemctl list-unit-files --type service
  $ sudo systemctl list-unit-files 'billing*'
  ```

- check if specific unit is enabled

  ```sh
  $ sudo systemctl is-enabled billing_prod.service
  ```

- start unit

  ```sh
  $ sudo systemctl start my-player.socket
  ```

- enable unit (start automatically on boot)

  ```sh
  $ sudo systemctl enable my-player.socket
  ```

- disable unit

  ```sh
  $ sudo systemctl disable my-player.socket
  ```

- remove unit

  ```sh
  $ sudo systemctl stop my-player.socket
  $ sudo systemctl disable my-player.socket
  $ sudo rm /etc/systemd/system/my-player.socket
  $ sudo systemctl daemon-reload
  $ sudo systemctl reset-failed
  ```

- poweroff or reboot system

  ```sh
  $ sudo systemctl poweroff
  $ sudo systemctl reboot
  ```

### journalctl

useful options:

- `-b, --boot` - show entries for specified boot
  (for current boot if argument is empty)
- `-e, --pager-end` - jump to the end of the journal
  (implies `-n1000`)
- `--no-tail` - show all lines
  (undoes `-e` and `-n` options)
- `-f, --follow` - follow journal (just like `tail -f`)
- `--since today` - show entries since specified today
  (in the format `"2012-10-30 18:17:16"` or special string like `today`)
- `-u, --unit` - show entries for specified unit only

NOTE: be cautious when using `-e` and `-n` options: they might cause some
      lines not to be printed.

view recent entries with explanation messages:

```sh
$ journalctl -xn
```

view journal for specified unit:

```sh
$ journalctl -u my-player.service
```

view journal for multiple units at once (entries are merged in chronological
order):

```sh
$ journalctl -u my-player.service -u my-player.socket
```

view journal for multiple units using wildcard:

```sh
$ journalctl -u my-*.service
```

view disk usage:

```sh
$ journalctl --disk-usage
```

creating custom unit file
-------------------------

- create _\<unit_name\>.\<type_extension\>_ file in _/etc/systemd/system_

  make sure it's a file - not symlink from somewhere else (even from your
  home directory). usually when unit is enabled systemd creates a symlink
  to unit file (located in _/etc/systemd/system_ for custom unit files)
  in _/etc/systemd/system/\<some_target\>.target.wants_ (as instructed in
  `[Install]` section of unit file).

  but if you create a symlink of unit file in _/etc/systemd/system_ (instead
  of copying file itself), systemd for some reason considers it to be enabled
  (it's not what you always want - maybe you want that unit to be started by
  another unit using `Requires` directive or it's a socket-activated service).

- enable unit if necessary

  ```sh
  $ sudo systemctl enable <unit_name>.<type_extension>
  ```

tips
----

### timer unit

- foo.service
- foo.timer

if using timer for associated service, enable timer unit only - associated
service unit will be activated when the timer is reached:

```sh
$ sudo systemctl enable foo.timer
```

### socket unit

- foo.service
- foo.socket

if using socket-based activation of the service, enable socket unit only -
associated service unit will be activated when anything is written to socket:

```sh
$ sudo systemctl enable foo.socket
```

debugging
---------

### view service logs

service units in most cases redirect their standard output and standard error
to systemd journal - this is where you, say, can find information why specific
service failed to start.

there are several options to view systemd journal:

- view all logs for specified unit

  ```sh
  $ sudo journalctl -u <service_name>
  ```

- *[RECOMMENDED]* view all logs for specified unit with explanations

  same as previous option but in most cases this is the most useful way to view
  journal because it provides maximum information about specified failed unit.

  ```sh
  $ sudo journalctl -xe -u <service_name>
  ```

- view terse status and most recent logs

  ```sh
  $ sudo systemctl status <service-name>
  ```
