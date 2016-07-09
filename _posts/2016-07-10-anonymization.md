---
layout: post
title: anonymization
date: 2016-07-10 01:17:59 +0300
access: private
categories: [security]
---

anonymization tips.

<!-- more -->

# sites to check anonymity and browser leaks

- <https://whoer.net/ru>
- <https://www.browserleaks.com/>

# chrome browser

## WebRTC

WebRTC protocol leaks your ISP address even when behind proxy or VPN server.

uBlock extensions allows to disable WebRTC but only when behind VPN server
(that is it doesn't work when using Browsec extension).

## extensions

- Browsec
- AdBlock
- Ghostery
- uBlock

## settings

### check `Send a "Do Not Track" request with your browsing traffic`

### disable flash (chrome://plugins)

### configure content settings (chrome://settings/content)

disable:

- location
- microphone
- camera
- unsandboxed plugin access
- midi devices full control
