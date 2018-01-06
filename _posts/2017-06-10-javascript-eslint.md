---
layout: post
title: JavaScript - ESLint
date: 2017-06-10 11:18:17 +0300
access: public
comments: true
categories: [js, eslint]
---

<!-- more -->

* TOC
{:toc}
<hr>

see also [JavaScript - Flow]({% post_url 2018-01-02-javascript-flow %}).

1. <https://github.com/eslint/eslint>
2. <https://objectcomputing.com/resources/publications/sett/january-2017-eslint-dont-write-javascript-without-it>

installation
------------

### install ESLint locally

```sh
$ yarn add eslint --dev
```

configuration
-------------

### generate ESLint config

```sh
$ $(npm bin)/eslint --init
...
Successfully created .eslintrc.yml file in <project-directory>
```

also ESLint config is required for syntastic to work (see below).

this will also install any additional plugins (based on your answers)
locally as development dependencies in _package.json_:

```javascript
"devDependencies": {
  ...
  "eslint": "^3.19.0",
  "eslint-plugin-react": "^7.0.1",
  ...
}
```

### set babel-eslint parser as default ESLint parser

1. <https://github.com/babel/babel-eslint/>

```sh
$ yarn add babel-eslint --dev
```

_.eslintrc.yml_:

```yaml
parser: 'babel-eslint'
```

standard ESLint parser marks some ES7/ES8/ES.Next syntax as invalid, say
[field declarations](https://github.com/tc39/proposal-class-fields):

```javascript
class Counter extends HTMLElement {
  x = 0;
}
```

### configure ESLint rules

1. <http://eslint.org/docs/user-guide/configuring>

extending `eslint:recommended` configuration enables subset of core rules
having a check mark on the [rules page](http://eslint.org/docs/rules/):

```yaml
extends:
  - 'eslint:recommended'
```

rule modifications are picked immediately on next syntastic run.
all the rules are added to `rules` section of _.eslintrc.yml_.

rule format: `name: setting` or `name: [setting, config]` where

- `setting` is one of `off` (0), `warn` (1) or `error` (2)
- `config` is rule-specific

some rules worth mentioning are listed below:

- enforce event handler naming conventions in JSX

  1. <https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-handler-names.md>

  ```yaml
  react/jsx-handler-names:
    - warn
    - eventHandlerPrefix: _handle
  ```

  this is considered okay:

  ```javascript
  <MyComponent onChange={this._handleChange} />
  <MyComponent onChange={this.props.onFoo} />
  ```

- mark variables used in JSX as used

  1. <https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-uses-vars.md>
  2. <https://github.com/eslint/eslint/issues/2283#issuecomment-260928189>

  > Since 0.17.0 the ESLint no-unused-vars rule does not detect variables used in JSX

  ```yaml
  react/jsx-uses-vars: error
  ```

  set rule ID to `error` (2) to turn it on.

  this rule has an effect only if `no-unused-rule` is enabled
  (enabled by default when extending `eslint:recommended`).

  still error is detected properly if JSX component is imported but not used.

- require space after function name in function declaration

  1. <http://eslint.org/docs/rules/space-before-function-paren>

  ```yaml
  space-before-function-paren:
    - warn
    - always
  ```

  see [JavaScript - Style Guide]({% post_url 2017-06-10-javascript-style-guide %})
  for description (section `space after function name in function declaration`).

usage
-----

### ignore ESLint errors on a one-off basis

```javascript
// eslint-disable-next-line
const foo = 123;
const foo = 123; // eslint-disable-line
```

### add ESLint script

_package.json_:

```json
"scripts": {
  "eslint": "eslint"
},
```

now ESLint can be run using any of these commands:

```sh
$ $(npm bin)/eslint
$ yarn run eslint
```

### run ESLint

```sh
$ yarn run eslint app/services/TeamHelpers.js
```

Vim integration
---------------

### syntastic

1. <https://medium.com/@hpux/vim-and-eslint-16fa08cc580f>
2. <http://remarkablemark.org/blog/2016/09/28/vim-syntastic-eslint/>
3. <https://github.com/vim-syntastic/syntastic/issues/1692>

NOTE: ESLint must be installed globally for
      syntastic `eslint` checker to be available.

syntastic error message when ESLint is not installed globally:

```
CacheErrors: Checker javascript/eslint is not available
```

```sh
$ yarn global add eslint
```

_~/.vim/vimrc_:

```vim
let g:syntastic_javascript_eslint_exe = '$(npm bin)/eslint'
let g:syntastic_javascript_checkers=['eslint']
```
