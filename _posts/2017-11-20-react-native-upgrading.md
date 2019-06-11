---
layout: post
title: React Native - Upgrading
date: 2017-11-20 11:33:05 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

react-native-git-upgrade
------------------------

1. <https://facebook.github.io/react-native/docs/upgrading.html>
2. <https://medium.zenika.com/easier-react-native-upgrades-with-rn-diff-5020b5c3de2d>
3. <https://github.com/facebook/react-native/issues/12261#issuecomment-284047679>
4. <https://github.com/facebook/react-native/issues/12261#issuecomment-286355163>

see [npm - Tips]({% post_url 2017-11-19-npm-tips %}) for `npm_reset`.

### unlink all packages

- using `upgrading-react-native`

  1. <https://github.com/npomfret/upgrading-react-native>

  ```sh
  $ npm_reset
  $ rm -rf ~/.gradle/caches/*
  $ git clone git@github.com:npomfret/upgrading-react-native.git
  $ node upgrading-react-native/unlink.js package.json
  $ sh unlink-packages.sh
  $ rm -rf unlink-packages.sh uninstall-packages.sh upgrading-react-native/
  ```

- using my own simple Ruby script

  _unlink\_all\_packages.rb_:

  ```ruby
  require 'json'

  hash = JSON.parse(File.read('package.json'))
  packages = hash['dependencies'].keys + hash['devDependencies'].keys

  packages.each.with_index(1) do |v, i|
    puts "(#{i}/#{packages.count}) unlinking #{v}"
    `react-native unlink #{v}`
  end
  ```

  ```sh
  $ npm_reset
  $ ruby unlink_all_packages.rb
  ```

### set merge strategy for project.pbxproj file

1. <https://github.com/facebook/react-native/issues/12261#issuecomment-341510929>
2. <http://roadfiresoftware.com/2015/09/automatically-resolving-git-merge-conflicts-in-xcodes-project-pbxproj/>

using `union` merge strategy for _project.pbxproj_ instructs
Git to use both sides (`ours` and `theirs`) during a merge.

_.gitattributes_:

```
*.pbxproj -text merge=union
```

### upgrade with react-native-git-upgrade

1. <https://facebook.github.io/react-native/docs/upgrading.html#upgrade-based-on-git>
2. <https://facebook.github.io/react-native/blog/2016/12/05/easier-upgrades.html>

```sh
$ npm install -g react-native-git-upgrade
$ react-native-git-upgrade
```

### resolve conflicts

resolve conflicts in all modified by `react-native-git-upgrade` files except
for _project.pbxproj_ - all conflicts should be merged automatically for this
file because of `union` merge strategy set in _.gitattributes_ previously.

enable gitgutter in MacVim and resolve conflicts in modified files manually
one by one (search for `ours`/`theirs` conflict markers).

### verify and fix project.pbxproj file

1. <https://github.com/Karumi/Kin>

check _project.pbxproj_ with Kin:

```sh
$ pip2 install kin
$ kin ios/iceperkapp.xcodeproj/project.pbxproj
```

NOTE: install Kin with `pip2` - not `pip3`.

if Python interpreter is not found - reinstall `kin`:

```sh
$ pip2 uninstall kin
$ pip2 install kin
```

### link all packages

```sh
$ react-native link
```

NOTE: this command didn't link all packages when upgrading to RN 0.52.1 -
      still it might be necessary to link some packages manually.

### repair CocoaPods

see the tip from [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}).

### post-upgrade instructions

```sh
$ npm_reset
$ react-native run-ios
$ react-native run-android
```

NOTE: don't forget to rebuild both iOS and Android applications!

***UPDATE***

after fixing different issues related to RN upgrade some libraries turned out
to be not linked - link all libraries again (just in case):

```sh
$ react-native link
```

even after running this command I had to link a lot of libraries manually
(see [React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %})) -
it looks like `react-native link` command is somehow broken in RN 0.52.1.

rn-diff-purge
-------------

1. <https://facebook.github.io/react-native/docs/upgrading.html#alternative>

I tried to upgrade RN from 0.52.1 to 0.59.9 with `rn-diff-purge`.

> <https://github.com/react-native-community/rn-diff-purge/>
>
> Why this repository?
>
> react-native-git-upgrade is painful. A simple diff by recreating the project
> is a much much simpler way to get a diff on every new React Native release.

### upgrade react and react-native dependencies

```sh
$ rm -rf node_modules
$ npm install --save react-native@0.59.9
...
npm WARN react-native@0.59.9 requires a peer of react@16.8.3 but none is installed. You must install peer dependencies yourself.
$ npm install --save react@16.8.3
```

### apply changes from diff by rn-diff-purge

1. <https://github.com/react-native-community/rn-diff-purge/blob/master/USAGE.md#recommended-method>
2. <https://github.com/react-native-community/rn-diff-purge/compare/release%2F0.52.1..release%2F0.59.9?diff=unified>

I did it manually - even for big files like _project.pbxproj_.

tips:

- try to add new lines - not to delete existing ones
- replace modified binary files too

### repair CocoaPods

see the tip from [React Native - iOS]({% post_url 2017-05-25-react-native-ios %}).

### try to build and start your application

```sh
$ npm install
$ react-native start
$ react-native run-android
$ react-native run-ios
```

most likely there will be lots of errors: many npm packages got broken because
of new versions of RN, Babel, Gradle, whatever.

see [React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %}),
`errors after upgrading RN from 0.52.1 to 0.59.9` section.
