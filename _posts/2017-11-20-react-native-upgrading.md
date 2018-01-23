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

1. <https://github.com/facebook/react-native/issues/12261#issuecomment-284047679>
2. <https://github.com/facebook/react-native/issues/12261#issuecomment-286355163>

unlink all packages
-------------------

1. <https://github.com/npomfret/upgrading-react-native>

```sh
$ watchman watch-del-all && rm -rf "$TMPDIR/react-*" && rm -rf node_modules/ && yarn cache clean && yarn install
$ rm -rf ~/.gradle/caches/*
$ git clone git@github.com:npomfret/upgrading-react-native.git
$ node upgrading-react-native/unlink.js package.json
$ sh unlink-packages.sh
$ rm -rf unlink-packages.sh uninstall-packages.sh upgrading-react-native/
```

or else it's possible to use a simple Ruby script _unlink\_all\_packages.rb_:

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
$ alias yarn_reset='watchman watch-del-all && rm -rf "$TMPDIR/react-*" && rm -rf node_modules/ && yarn cache clean && yarn install'
$ yarn_reset
$ ruby unlink_all_packages.rb
```

set merge strategy for project.pbxproj file
-------------------------------------------

1. <https://github.com/facebook/react-native/issues/12261#issuecomment-341510929>
2. <http://roadfiresoftware.com/2015/09/automatically-resolving-git-merge-conflicts-in-xcodes-project-pbxproj/>

using `union` merge strategy for _project.pbxproj_ file instructs
Git to use both sides (`ours` and `theirs`) during a merge.

_.gitattributes_:

```
*.pbxproj -text merge=union
```

upgrade with react-native-git-upgrade
-------------------------------------

1. <https://facebook.github.io/react-native/docs/upgrading.html>

```sh
$ yarn global add react-native-git-upgrade
$ react-native-git-upgrade
```

resolve conflicts
-----------------

resolve conflicts in all modified files except for _project.pbxproj_ -
all conflicts should be merged automatically for this file because of
`union` merge strategy set in _.gitattributes_ previously.

enable gitgutter in MacVim and resolve conflicts in changed files manually
one by one (search for `ours`/`theirs` conflict markers).

verify and fix project.pbxproj file
-----------------------------------

1. <https://github.com/Karumi/Kin>

check _project.pbxproj_ with Kin:

```sh
$ pip2 install kin
$ kin ios/iceperkapp.xcodeproj/project.pbxproj
```

NOTE: use `pip2` - not `pip3`.

link all packages
-----------------

```sh
$ react-native link
```

reinstall CocoaPods
-------------------

I had to reinstall CocoaPods due to several CocoaPods related issues - see
[React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %}).

post-upgrade instructions
-------------------------

```sh
$ yarn_reset
$ react-native run-ios
$ react-native run-android
```

NOTE: don't forget to rebuild applications for both iOS and Android!

**UPDATE**

after fixing different issues related to RN upgrade some libraries
turned out to be not linked - try to link them again (just in case):

```sh
$ react-native link
```

even after running this command I had to link a lot of libraries manually -
it looks like `react-native link` command is somehow broken in RN 0.52.1
(see [React Native - Troubleshooting]({% post_url 2017-07-04-react-native-troubleshooting %})).
