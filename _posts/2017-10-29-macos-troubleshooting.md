---
layout: post
title: macOS - Troubleshooting
date: 2017-10-29 01:42:13 +0300
access: public
comments: true
categories: [macos]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

## battery percentage is not properly updating

1. <https://support.apple.com/en-us/HT201295>

reset the SMC.

## active developer directory is a CLT instance

> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer
> directory '/Library/Developer/CommandLineTools' is a command line tools
> instance

1. <http://stackoverflow.com/questions/17980759#17980786>

just switch to Xcode directory:

```sh
$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

## unable to update Xcode from 8.2.1 on El Capitan

1. <https://stackoverflow.com/questions/43065475/unable-to-update-to-xcode-8-3>

Xcode 8.3 and higher requires macOS Sierra.

## tab navigation doesn't work in Google Chrome

OS-wide tab navigation shorcuts (`<D-S-[>`, `<D-S-[>`) no longer work after
upgrading Google Chrome to 69.0.3497.100 and only when using Dvorak keyboard
layout - it's like okay again after switching to Qwerty.

**solution**

1. <https://superuser.com/a/1260437/326775>

| Preferences                               |
| ----------------------------------------- |
| Keyboard → Shortcuts (tab) → App Shorcuts |

add these shortcuts:

- Google Chrome:
  - Select Previous Tab: `<D-S-[>`
  - Select Next Tab: `<D-S-[>`

**_UPDATE_**

this issue is fixed in Google Chrome 70.0.3538.67 - it's possible to remove
custom shortcuts now.

## macOS won't display Wi-Fi login screen

I have the same problem on iPad too.

**solution**

1. <https://apple.stackexchange.com/questions/211430>

the culprit is Google DNS servers: by default Wi-Fi network tries to use its own
DNS servers to load login screen (which are accessible a priori). instead in my
case it's forced to use Google DNS servers (1.1.1.1 and 8.8.8.8) which are not
yet available.

solution is to remove Google DNS servers altogether so that Wi-Fi network's
default DNS servers are used.

if it's all okay with DNS servers try to open:

- magic URL <http://captive.apple.com/hotspot-detect.html>
- Wi-Fi router admin panel (192.168.0.1, 192.168.1.1, etc.)
- any site URL

in all cases you should be redirected to Wi-Fi login screen.

## Dashlane window is not visible

**solution**

most likely Dashlane has already logged me out but login pop-up window is not
shown - it doesn't matter how many times I open and close application itself.

solution is to kill `DashlaneAgent` process - say, using `htop` or
`Activity Monitor`. login pop-up window must appear the next time Dashlane is
launched.

## youtube-dl is unable to download videos

```
$ youtube-dl <URL>
...
ERROR: unable to download video data: HTTP Error 403: Forbidden
```

**solution**

1. <https://www.ostechnix.com/fix-unable-to-download-video-data-http-error-403-forbidden-error/>

```sh
$ youtube-dl --rm-cache-dir
```
