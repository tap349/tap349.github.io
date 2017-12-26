---
layout: post
title: macOS - Troubleshooting
date: 2017-10-29 01:42:13 +0300
access: public
comments: true
categories: [macos]
---

<!-- more -->

* TOC
{:toc}
<hr>

battery percentage is not properly updating
-------------------------------------------

reset the SMC:

<https://support.apple.com/en-us/HT201295>

active developer directory is a CLT instance
--------------------------------------------

> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer
> directory '/Library/Developer/CommandLineTools' is a command line tools instance

<http://stackoverflow.com/questions/17980759#17980786>

just switch to Xcode directory:

```sh
$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

unable to update Xcode from 8.2.1 on El Capitan
-----------------------------------------------

1. <https://stackoverflow.com/questions/43065475/unable-to-update-to-xcode-8-3>

Xcode 8.3 and higher requires macOS Sierra.
