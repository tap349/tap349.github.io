---
layout: post
title: JavaScript - Flow
date: 2018-01-02 02:28:57 +0300
access: public
comments: true
categories: [js, eslint, flow]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://flow.org/en/docs/getting-started/>
2. <https://github.com/facebook/flow>
3. <https://github.com/ryyppy/flow-guide>

installation
------------

- install Flow globally

  ```sh
  $ brew install flow
  ```

- install Babel preset for Flow plugins

  ```sh
  $ yarn add babel-preset-flow --dev
  ```

  add preset to _.babelrc_:

  ```
  {
    "presets": ["flow"]
  }
  ```

  > Now, Babel will always strip away Flow source and your JS runtime can
  > interpret the code. This is especially important for feeding into ESlint.

- install eslint-plugin-flowtype ESLint plugin (?)

  it looks like this plugin provides Flow linting rules for
  ESLint to be able to use ESLint to find Flow type errors
  (`yarn run eslint` command or syntastic `eslint` checker)
  but it's not the case: type errors are found only when using
  `flow status` command or syntastic `flow` checker.

  <https://github.com/ryyppy/flow-guide#eslint-configuration>:

  > there will be some warnings about unused variables whenever you
  > write type declarations. The eslint plugin eslint-plugin-flowtype
  > will mute those warnings.

  I don't see any warnings this plugin is supposed to mute either.

  => I have removed this plugin so far.

usage
-----

- prepare project

  ```sh
  $ flow init
  ```

  this will create empty _.flowconfig_ at project root.

- flag files to be checked with `// @flow`

  this flag can be placed anywhere in JS file -
  it just instructs Flow to check this file.

- run Flow

  ```sh
  # same as `flow status` - starts Flow server (if not started yet),
  # does a full check, prints the results and monitors changes to
  # your code in the background checking those changes incrementally
  # for errors - current errors can be shown by typing `flow` again
  $ flow
  # does a full check in the foreground without starting Flow server
  $ flow check
  ```

Vim integration
---------------

- syntastic

  _~/.vim/vimrc_:

  ```vim
  let g:syntastic_javascript_checkers = ['eslint', 'flow']
  ```

- `vim-flow` plugin

  syntastic already performs linting of JS files so linting
  provided by this plugin (automatic or manual checks - via
  `FlowMake` command) seems to be superfluous.

  still `FlowJumpToDef` and `FlowType` commands might be useful -
  just make sure to disable automatic checks on save!

style guide
-----------

1. <https://github.com/ryyppy/flow-guide#flow-style-guide>
