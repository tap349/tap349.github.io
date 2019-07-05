---
layout: post
title: anonymization
date: 2016-07-10 01:17:59 +0300
access: private
categories: [security]
---

anonymization tips.

<!-- more -->

sites to check anonymity and browser leaks
------------------------------------------

- <https://whoer.net>
- <https://ipleak.net>
- <http://iknowwhatyoudownload.com>
- <https://www.browserleaks.com>
- <https://2ip.ru>

chrome browser
--------------

### WebRTC

WebRTC protocol leaks your ISP address even when behind proxy or VPN server.

uBlock extensions allows to disable WebRTC but it doesn't seem to work -
even when using VPN <https://whoer.net/ru> shows that WebRTC is enabled but
local IP address is no longer available (unlike when using Browsec extension).

### extensions

- Browsec
- AdBlock
- Ghostery
- uBlock

### settings

#### leave only English language (chrome://settings/languages)

#### check `Send a "Do Not Track" request with your browsing traffic`

#### disable flash (chrome://plugins)

#### configure content settings (chrome://settings/content)

disable:

- location
- microphone
- camera
- unsandboxed plugin access
- midi devices full control

VPN
---

### use DNS servers not bound to specific country

- 8.8.8.8, 8.8.4.4 (Google's DNS)
- 208.67.222.222, 208.67.220.220 (OpenDNS)

### disable IPv6

```sh
$ networksetup -setv6off Wi-Fi
```
