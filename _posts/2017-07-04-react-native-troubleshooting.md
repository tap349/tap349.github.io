---
layout: post
title: React Native - Troubleshooting
date: 2017-07-04 18:40:40 +0300
access: public
categories: [react-native]
---

<!-- more -->

## errors after upgrading RN

```sh
$ npm start

...
> node node_modules/react-native/local-cli/cli.js start

module.js:487
    throw err;
    ^

Error: Cannot find module 'xmldoc'
    at Function.Module._resolveFilename (module.js:485:15)
```

and similar errors when some module cannot be found.

solution:

use yarn instead of npm:

```sh
$ rm package-lock.json
$ rm -rf node-modules/
$ yarn
$ npm start
```
