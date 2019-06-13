---
layout: post
title: npm/Yarn - Tips
date: 2017-11-19 03:09:25 +0300
access: public
comments: true
categories: [npm, yarn, js]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) install package globally
---------------------------------

### npm

```sh
$ npm install -g semistandard
/usr/local/bin/semistandard -> /usr/local/lib/node_modules/semistandard/bin/cmd.js
$ which semistandard
/usr/local/bin/semistandard
```

- global packages dir: _/usr/local/lib/node_modules/_

  ```sh
  $ npm -g root
  /usr/local/lib/node_modules/
  ```

### Yarn

```sh
$ yarn global add semistandard
$ which semistandard
/usr/local/bin/semistandard
```

- global packages dir: _~/.config/yarn/global/_
- global package.json: _~/.config/yarn/global/package.json_
- global _yarn.lock_: _~/.config/yarn/global/yarn.lock_

(how to) reinstall all packages
-------------------------------

### npm

```sh
$ rm -rf node_modules && npm install
```

### Yarn

```sh
$ rm -rf node_modules && yarn
```

(how to) remove not used packages
---------------------------------

### npm

not used packages appear after removing packages from _package.json_ manually.

```sh
$ npm prune
```

### Yarn

`yarn install` command removes not used packages as well.

(how to) run script or locally installed executable
---------------------------------------------------

- executables

  executables are binaries provided by npm packages.

  they are linked into _node\_modules/.bin/_:

  > <https://docs.npmjs.com/files/folders#executables>
  >
  > executables are linked into ./node_modules/.bin so that they can be made
  > available to scripts run through npm.

  it's possible to run executables directly (npm bin folder is returned by
  `npm bin` command):

  ```sh
  $ $(npm bin)/flow
  ```

- scripts

  scripts are entries in `scripts` section of _package.json_, they are used
  to run executables (probably with custom arguments).

### npm

```sh
$ npm run-script <bin_or_script>
```

### Yarn

```sh
$ yarn run <bin_or_script>
```

(how to) reset all (and clean cache)
------------------------------------

1. <https://gist.github.com/jarretmoses/c2e4786fd342b3444f3bc6beff32098d>

> message from `npm start`:
>
> 1. Clear watchman watches: `watchman watch-del-all`.
> 2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
> 3. Reset Metro Bundler cache: `rm -rf $TMPDIR/react-*` or `npm start -- --reset-cache`.
> 4. Remove haste cache: `rm -rf $TMPDIR/haste-map-react-native-packager-*`.

it might be necessary to clean cache, say, when build fails. in many cases it's
enough just to reset cache => try `npm start --reset-cache` command first.

### npm

```zsh
# _~/.zshenv_

alias npm_reset='\
  watchman watch-del-all &&
  rm -rf "$TMPDIR/react-native-packager-cache-*" &&
  rm -rf "$TMPDIR/metro-bundler-cache-*" &&
  rm -rf "$TMPDIR/haste-map-react-native-packager-*" &&
  rm -rf node_modules &&
  npm cache clean --force &&
  npm install
  '
```

```sh
$ npm_reset
$ npm start --reset-cache
```

### Yarn

```zsh
# _~/.zshenv_

alias yarn_reset='\
  watchman watch-del-all &&
  rm -rf "$TMPDIR/react-native-packager-cache-*" &&
  rm -rf "$TMPDIR/metro-bundler-cache-*" &&
  rm -rf "$TMPDIR/haste-map-react-native-packager-*" &&
  rm -rf node_modules &&
  yarn cache clean &&
  yarn install
  '
```

```sh
$ yarn_reset
$ yarn start --reset-cache
```

(how to) update a package
-------------------------

### npm

update to the latest version allowed by semantic version in _package.json_:

```sh
$ npm update foo
```

install specific version:

```sh
$ npm install --save foo@1.2.3
```

### Yarn

update to the latest version allowed by semantic version in _package.json_:

```sh
$ yarn upgrade foo
```

install specific version:

```sh
$ yarn install foo@1.2.3
```
