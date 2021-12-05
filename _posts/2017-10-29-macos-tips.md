---
layout: post
title: macOS - Tips
date: 2017-10-29 01:49:12 +0300
access: public
comments: true
categories: [macos]
---

<!-- more -->

* TOC
{:toc}
<hr>

open application in terminal
----------------------------

- in the background

  ```sh
  $ open MyApp
  ```

- in the foreground

  ```sh
  $ /Applications/MyApp.app/Contents/MacOS/MyApp
  ```

mount NTFS volume in read-write mode
------------------------------------

1. <https://github.com/osxfuse/osxfuse/wiki/NTFS-3G>

```sh
$ brew cask install osxfuse
$ brew install ntfs-3g
$ sudo reboot
$ diskutil list
$ sudo mkdir /Volumes/NTFS
$ sudo ntfs-3g /dev/disk2s1 /Volumes/NTFS -olocal -oallow_other
```

or else use Disk Utility application to find device identifier.
