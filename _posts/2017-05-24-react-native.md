---
layout: post
title: React Native
date: 2017-05-24 16:27:38 +0300
access: public
categories: [react-native]
---

<!-- more -->

## installation and running

- <https://facebook.github.io/react-native/docs/getting-started.html>
- <https://docs.npmjs.com/getting-started/installing-npm-packages-globally>

## troubleshooting

### cannot uninstall android-sdk

<https://github.com/caskroom/homebrew-cask/issues/32139>

```sh
$ brew cask uninstall android-sdk
Error: No such file or directory - /usr/local/share/android-sdk
```

solution:

```sh
$ ln -s /usr/local/Caskroom/android-sdk/25.2.3 /usr/local/share/android-sdk
$ brew cask uninstall android-sdk
```

### warning when running tools from android-sdk

<https://askubuntu.com/questions/885658>

```sh
$ sdkmanager --list                                                                                                                tap@MacBook-Pro-Personal
Warning: File /Users/tap/.android/repositories.cfg could not be loaded.
```

solution:

```sh
$ touch ~/.android/repositories.cfg
```
