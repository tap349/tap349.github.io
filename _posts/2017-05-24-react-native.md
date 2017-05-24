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

### android

- <https://medium.com/skyshidigital/install-react-native-without-android-studio-366317419e7e>
- <https://stackoverflow.com/questions/42718973/run-avd-emulator-without-android-studio>

```sh
$ brew install node watchman
$ npm install -g react-native-cli
$ brew cask install java
```

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

### truncated package paths in output from sdkmanager

<https://stackoverflow.com/questions/42460205>

```sh
$ sdkmanager --list
...
Available Packages:
  Path                              | Version      | Description
  -------                           | -------      | -------
  add-ons;addon-g..._apis-google-15 | 3            | Google APIs
  add-ons;addon-g..._apis-google-16 | 4            | Google APIs
```

solution:

```sh
$ sdkmanager --list --verbose
```
