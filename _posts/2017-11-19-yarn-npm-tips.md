---
layout: post
title: Yarn/npm - Tips
date: 2017-11-19 03:09:25 +0300
access: public
comments: true
categories: [yarn, npm, js]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://yarnpkg.com/lang/en/docs/migrating-from-npm/>

(how to) install package globally
---------------------------------

### npm

```sh
$ npm install semistandard -g
/usr/local/bin/semistandard -> /usr/local/lib/node_modules/semistandard/bin/cmd.js
$ which semistandard
/usr/local/bin/semistandard
```

effective _node\_modules/_ global directory:

```sh
$ npm root -g
/usr/local/lib/node_modules/
```

### Yarn

```sh
$ yarn global add rnpm
$ which rnpm
/usr/local/bin/rnpm
```

- global packages are stored in _~/.config/yarn/global/_
- global packages are saved to _~/.config/yarn/global/package.json_
  (only when installed with Yarn)

(how to) reinstall all packages
-------------------------------

```sh
$ rm -rf node_modules && yarn install
$ rm -rf node_modules && npm install
```

[npm] (how to) remove not used packages
---------------------------------------

not used packages appear after removing packages from _package.json_ manually.

```sh
$ npm prune
```

dependencies vs. development dependencies
-----------------------------------------

install package as dependency:

```sh
$ npm install <package> --save
$ yarn add <package>
```

install package as development dependency:

```sh
$ npm install <package> --save-dev
$ yarn add <package> --dev
```

installing package as dependency saves it in `"dependencies"` section
of _package.json_ (application cannot run without it) while installing
package as development dependency saves it in `"devDependencies"` section
(packages are used in development environment only - say, in test suites
or by transpilers).

(how to) run script or locally installed executable
---------------------------------------------------

`$(npm bin)` resolves to _node\_modules/.bin/_ - it can be used to run
executables from this directory directly.

- locally installed executables are stored in _node\_modules/.bin/_
  (= symlinks to JS executable files provided by npm packages)
- scripts are defined in `scripts` section of _package.json_
  (= usually invocations of these executables with custom arguments)

`yarn run` runs both of them:

```sh
$ yarn run <bin_or_script>
$ npm run <bin_or_script>
$ npm run-script <bin_or_script>
```

(how to) reset all (and clean cache)
------------------------------------

1. <https://gist.github.com/jarretmoses/c2e4786fd342b3444f3bc6beff32098d>

_~/.zshenv_:

```zsh
alias yarn_reset='watchman watch-del-all && rm -rf "$TMPDIR/react-*" && rm -rf node_modules && yarn cache clean && yarn install'
```

```sh
$ yarn_reset
$ yarn start --reset-cache
```

it might be necessary to clean cache, say, when build fails.

if it's necessary to reset cache only, using `yarn start --reset-cache`
might suffice - try it first.

(how to) update a package
-------------------------

- change package version in _package.json_
- `yarn upgrade <package>`

if Yarn/npm refuses to update a package (maybe it's a hack):

- change package version in _package.json_
- `yarn install`/`npm install`

don't use peer dependencies
---------------------------

1. <https://lexi-lambda.github.io/blog/2016/08/24/understanding-the-npm-dependency-model/>

do I need to use peer dependencies if I am not npm package author?

say, _node_modules/react-native-push-notification/package.json_:

```json
"peerDependencies": {
  "react-native": ">=0.33"
},
```

so `react-native-push-notification` peer-depends on `react-native` -
what would happen if I use `react-native` of unsupported version:

1. as a depedency of my application?
2. as a peer dependency of my application?

- using `react-native` of unsupported version as a dependency

  _package.json_:

  ```json
  "dependencies": {
    "react-native": "0.30.0",
    "react-native-push-notification": "^3.0.0"
  },
  ```

  ```sh
  $ npm install
  ...
  npm WARN react-native-push-notification@3.0.1 requires a peer of react-native@>=0.33 but none was installed.
  ```

  => `react-native@0.30.0` is installed while a peer dependency of
  `react-native-push-notification` is not.

- using `react-native` of unsupported version as a peer dependency

  _package.json_:

  ```json
  "peerDependencies": {
    "react-native": "0.30.0"
  },
  "dependencies": {
    "react-native-push-notification": "^3.0.0"
  },
  ```

  ```sh
  $ npm install
  npm WARN my_app@0.0.1 requires a peer of react-native@0.30.0 but none was installed.
  npm WARN react-native-push-notification@3.0.1 requires a peer of react-native@>=0.33 but none was installed.
  ```

  => `react-native` is not installed at all - even I remove
  `react-native-push-notification` from the list of dependencies
  (no packages depend on specific version of `react-native` any more).

TL;DR:

don't use peer dependencies if you are not npm package author -
they are not automatically installed:

> peer dependencies are not automatically installed unless
> a dependent package explicitly depends on the peer package itself

so if you application depends on `react-native` package,
add it to `depedencies` along with its version.

another package (say, `react-native-foo`) that has `react-native`
as a peer dependency will be installed only if version constraints
of peer dependency are met by currently installed `react-native`.
