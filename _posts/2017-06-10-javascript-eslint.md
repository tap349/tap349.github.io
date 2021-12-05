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

### install `babel-eslint`

1. <https://github.com/babel/babel-eslint/>

```sh
$ yarn add babel-eslint --dev
```

use `babel-eslint` as default ESLint parser:

```yaml
# .eslintrc.yml

parser: 'babel-eslint'
```

standard ESLint parser marks some ES7/ES8/ES.Next syntax as invalid, say
[field declarations](https://github.com/tc39/proposal-class-fields):

```javascript
class Counter extends HTMLElement {
  x = 0;
}
```

### install `eslint-plugin-react`

NOTE: this plugin will be installed automatically if you generate ESLint
      config from anew and opt for React support.

install this plugin if you use `react` plugin in ESLint config:

```sh
$ yarn add eslint-plugin-react --dev
```

### install `eslint-plugin-flowtype`

NOTE: this plugin will be installed automatically if you generate ESLint
      config from anew and opt for Flow type support.

install this plugin if you use `flowtype` plugin in ESLint config:

```sh
$ yarn add eslint-plugin-flowtype --dev
```

see [JavaScript - Flow]({% post_url 2018-01-02-javascript-flow %}) on how
to configure this plugin.

configuration
-------------

### generate ESLint config

```sh
$ yarn run eslint --init
...
Successfully created .eslintrc.yml file in <project-directory>
```

also ESLint config is required for syntastic to work (see below).

this will also install any additional plugins (based on your answers)
locally as development dependencies in _package.json_:

```javascript
"devDependencies": {
  // ...
  "eslint": "^3.19.0",
  "eslint-plugin-react": "^7.0.1",
  // ...
}
```

### configure ESLint rules

1. <http://eslint.org/docs/user-guide/configuring>

extending `eslint:recommended` configuration enables subset of core rules
having a check mark on the [rules page](http://eslint.org/docs/rules/):

```yaml
# .eslintrc.yml

extends:
  - 'eslint:recommended'
```

rule modifications are picked immediately on next syntastic run.
all the rules are added to `rules` section of _.eslintrc.yml_.

rule format: `name: setting` or `name: [setting, config]` where

- `setting` is one of `off` (0), `warn` (1) or `error` (2)
  (prefer 1 instead of `warn` or `error` for `foo-uses-bar` rules)
- `config` is rule-specific

some rules worth mentioning are listed below:

- enforce event handler naming conventions in JSX

  1. <https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-handler-names.md>

  ```yaml
  # .eslintrc.yml

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
  # .eslintrc.yml

  react/jsx-uses-vars: error
  ```

  set rule ID to `error` (2) to turn it on.

  this rule has an effect only if `no-unused-rule` is enabled
  (enabled by default when extending `eslint:recommended`).

  still error is detected properly if JSX component is imported but not used.

- require space after function name in function declaration

  1. <http://eslint.org/docs/rules/space-before-function-paren>

  ```yaml
  # .eslintrc.yml

  space-before-function-paren:
    - warn
    - always
  ```

  see [JavaScript - Style Guide]({% post_url 2017-06-10-javascript-style-guide %})
  for description (section `space after function name in function declaration`).

usage
-----

### ignore ESLint errors on a one-off basis

1. <https://github.com/eslint/eslint/issues/6903#issuecomment-239646089>

```javascript
// eslint-disable-next-line
const foo = 123;
const foo = 123; // eslint-disable-line

render () {
  /* eslint-disable react/jsx-handler-names */
  return (
    <ReactTags
      tags={this.props.tags}
      handleAddition={this.props.onAdd}
      handleDelete={this.props.onRemove}
    />
  );
  /* eslint-enable react/jsx-handler-names */
}
```

### run ESLint

```sh
$ yarn run eslint file.js
```

Vim integration
---------------

### [NOT USED] syntastic

1. <https://medium.com/@hpux/vim-and-eslint-16fa08cc580f>
2. <http://remarkablemark.org/blog/2016/09/28/vim-syntastic-eslint/>
3. <https://github.com/vim-syntastic/syntastic/issues/1692>

NOTE: ESLint must be installed globally for syntastic `eslint` checker
      to be available (like for Flow).

syntastic error message when ESLint is not installed globally:

```
CacheErrors: Checker javascript/eslint is not available
```

```sh
$ yarn global add eslint
```

```vim
" ~/.vim/vimrc

let g:syntastic_javascript_eslint_exe = '$(npm bin)/eslint'
let g:syntastic_javascript_checkers=['eslint']
```

### ale

unlike syntastic, ALE doesn't require ESLint to be installed globally -
it finds ESLint script (`$(npm bin)/eslint`) somehow.

```vim
" ~/.vim/vimrc

let g:ale_linters = {
      \   'javascript': ['eslint']
      \ }
```
