---
layout: post
title: JavaScript - ESLint
date: 2017-06-10 11:18:17 +0300
access: public
comments: true
categories: [js, eslint]
---

<!-- more -->

1. <https://github.com/eslint/eslint>
2. <https://medium.com/@hpux/vim-and-eslint-16fa08cc580f>
3. <http://remarkablemark.org/blog/2016/09/28/vim-syntastic-eslint/>

## install ESLint globally

```sh
$ yarn global add eslint
```

it's required to generate initial ESLint config for specific project
and for syntastic to work (see below).

## generate ESLint config

```sh
$ cd <project-directory>
$ $(npm bin)/eslint --init
...
Successfully created .eslintrc.yml file in <project-directory>
```

this will install both _eslint_ package and additional plugins (based
on your answers) locally as development dependencies in _package.json_:

```javascript
"devDependencies": {
  ...
  "eslint": "^3.19.0",
  "eslint-plugin-react": "^7.0.1",
  ...
}
```

## use babel-eslint parser as default ESLint parser

1. <https://github.com/babel/babel-eslint/>

```sh
$ npm install babel-eslint --save-dev
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

## configure ESLint rules

1. <http://eslint.org/docs/user-guide/configuring>

extending `eslint:recommended` configuration enables subset of core rules
having a check mark on the [rules page](http://eslint.org/docs/rules/):

```yaml
extends: 'eslint:recommended'
```

rule modifications are picked immediately on next syntastic run.
all the rules are added to `rules` section of _.eslintrc.yml_.

some rules worth mentioning are listed below:

- enforce event handler naming conventions in JSX

  1. <https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-handler-names.md>

  ```yaml
  react/jsx-handler-names: warn
  ```

  this is considered okay:

  ```javascript
  <MyComponent onChange={this.handleChange} />
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
  (enabled by default if extending `eslint:recommended`).

  still error is detected properly if JSX component is imported but not used.

- require space after function name in function declaration

  1. <http://eslint.org/docs/rules/space-before-function-paren>

  ```yaml
  space-before-function-paren:
    - warn
    - always
  ```

  see [JavaScript - StyleGuide]({% post_url 2017-06-10-javascript-style-guide %})
  for description (section `space after function name in function declaration`).

## configure syntastic

1. <https://github.com/vim-syntastic/syntastic/issues/1692>

```vim
let g:syntastic_javascript_eslint_exe = '$(npm bin)/eslint'
let g:syntastic_javascript_checkers=['eslint']
```

so the only combination that worked for me (both conditions are mandatory):

- install ESLint globally (for checker to be enabled by syntastic)
- install ESLint locally (`eslint --init` does it) and specify path to
  local `eslint` executable as the value of `g:syntastic_javascript_eslint_exe`
