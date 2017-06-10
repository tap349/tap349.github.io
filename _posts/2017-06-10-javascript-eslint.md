---
layout: post
title: JavaScript - ESLint
date: 2017-06-10 11:18:17 +0300
access: public
categories: [js, eslint]
---

<!-- more -->

- <https://github.com/eslint/eslint>
- <https://medium.com/@hpux/vim-and-eslint-16fa08cc580f>
- <http://remarkablemark.org/blog/2016/09/28/vim-syntastic-eslint/>

## install ESLint globally

```sh
$ npm install eslint --global
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

```sh
$ npm install babel-eslint --save-dep
```

_.eslintrc.yml_:

```yaml
parser: 'babel-eslint'
```

standard ESLint parser marks some ES7 syntax as invalid, say
[field declarations](https://github.com/tc39/proposal-class-fields):

```javascript
class Counter extends HTMLElement {
  x = 0;
}
```

## tweak ESLint config (_.eslintrc.yml_)

restart Vim for these changes to take effect.

- 4 spaces are used for indentation by default - change to 2 spaces:

  ```yaml
  rules:
    indent:
      - error
      - 2
  ```

- disable `no-unused-vars` for imported and used JSX components

  - <https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-uses-vars.md>
  - <https://github.com/eslint/eslint/issues/2283#issuecomment-260928189>

  ```yaml
  rules:
    react/jsx-uses-vars:
      - error
  ```

  set the rule to `error` (2) is to enable it.

## configure syntastic

<https://github.com/vim-syntastic/syntastic/issues/1692>

```vim
let g:syntastic_javascript_eslint_exe = '$(npm bin)/eslint'
let g:syntastic_javascript_checkers=['eslint']
```

so the only combination that worked for me (both conditions are mandatory):

- install ESLint globally (for checker to be enabled by syntastic)
- install ESLint locally (`eslint --init` does it) and specify path
  to local `eslint` executable in `g:syntastic_javascript_eslint_exe`
