---
layout: post
title: npm/Yarn - Troubleshooting
date: 2018-05-03 21:25:56 +0300
access: public
comments: true
categories: [npm, yarn, js]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

The engine "node" is incompatible with this module
--------------------------------------------------

```
$ yarn install
...[2/4] 🚚  Fetching packages...
error upath@1.0.4: The engine "node" is incompatible with this module. Expected version ">=4 <=9".
error Found incompatible module
```

**solution**

1. <https://github.com/gilbarbara/react-joyride/issues/131#issuecomment-253319816>

```sh
$ yarn --ignore-engines
```

npm doesn't install the version of dependency from package-lock.json
--------------------------------------------------------------------

this can happen if you tried to install that depedency explicitly beforehand
(say, by adding it to `dependencies` or `devDependencies` in _package.json_)
but uninstalled it in the end. still incorrect version of that depedency (the
one you specified in _package.json_) is resolved over and over again.

**solution**

clean npm cache:

```sh
$ npm cache clean --force
$ npm install
```
