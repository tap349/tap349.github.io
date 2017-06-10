---
layout: post
title: JavaScript - ESLint
date: 2017-06-10 11:18:17 +0300
access: public
categories: [js, eslint]
---

<!-- more -->

<https://github.com/eslint/eslint>

- install ESLint globally

  ```sh
  $ npm install eslint --global
  ```

- generate ESLint config

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

  by default 4 spaces are used for indentation - change to 2 spaces.

- configure syntastic

  <https://github.com/vim-syntastic/syntastic/issues/1692>

  ```vim
  let g:syntastic_javascript_eslint_exe = '$(npm bin)/eslint'
  let g:syntastic_javascript_checkers=['eslint']
  ```

  so the only combination that worked for me (both conditions are mandatory):

  - install ESLint globally (for checker to be enabled by syntastic)
  - install ESLint locally (`eslint --init` does it) and specify path
    to local `eslint` executable in `g:syntastic_javascript_eslint_exe`
