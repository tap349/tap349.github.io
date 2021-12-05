---
layout: post
title: Steam - Troubleshooting
date: 2018-10-07 21:20:48 +0300
access: public
comments: true
categories: [steam]
---

<!-- more -->

* TOC
{:toc}
<hr>

Steam application hangs on startup
----------------------------------

A single window is shown without any text on it and any signs of what is
going on.

**solution**

1. <https://steamcommunity.com/discussions/forum/2/1698294337771601383?ctp=3#c1738841319813048802>
2. <https://www.dssw.co.uk/reference/diskutil.html>

it has turned out Steam doesn't play well with case-sensitive filesystems
on macOS - doesn't play at all to be precise.

one of the solutions is to create a separate case-insensitive APFS volume
and move all Steam data (_~/Library/Application Support/Steam/_) to it:

```sh
$ diskutil apfs addVolume disk1 APFS Steam
Exporting new APFS Volume "Steam" from APFS Container Reference disk1
Started APFS operation on disk1
Preparing to add APFS Volume to APFS Container disk1
Creating APFS Volume
Created new APFS Volume disk1s5
Mounting APFS Volume
Setting volume permissions
Disk from APFS operation: disk1s5
Finished APFS operation on disk1
$ mv ~/Library/Application\ Support/Steam /Volumes/Steam/Library
$ ln -s /Volumes/Steam/Library ~/Library/Application\ Support/Steam
```
