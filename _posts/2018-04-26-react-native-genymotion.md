---
layout: post
title: React Native - Genymotion
date: 2018-04-26 14:13:16 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

configuration
-------------

### install GApps

1. <https://stackoverflow.com/a/20137324/3632318>

| emulator (app): right toolbar → `Open GAPPS` (button)

GApps are required to receive pushes (see `troubleshooting` section).

### virtual keyboard

it's enabled by default on Android Emulator.

| Genymotion: `Configure this virtual device` (tool icon button)

- [x] `Use virtual keyboard for text input`

debugging
---------

### Developer Menu

| emulator (app): `<D-m>`

or

| emulator (app): right toolbar → `Menu` (drawer icon button)

### connect to local web server

1. <https://stackoverflow.com/a/20257547/3632318>

use `10.0.3.2:3000`.

troubleshooting
---------------

### reloading doesn't work

network requests are sent to development server after running
application with `react-native run-android` but are no longer
sent after the first reload (connection with packager is not lost).

**solution**

install GApps (see `configuration` section).

temporary fixes:

- run application again with `react-native run-android`
- force close and open application inside emulator

***UPDATE***

reloading doesn't work again so temporary fixes are still pertinent.

### pushes are not received

no push token is associated with user device (just like on iOS emulator).

**solution**

install GApps (see `configuration` section).

NOTE: it might be required to restart emulator and run application again
      for pushes to work.
