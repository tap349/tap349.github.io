---
layout: post
title: JavaScript - Style Guide
date: 2017-06-10 18:36:47 +0300
access: public
comments: true
categories: [js]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://standardjs.com/>
2. <http://javascript.crockford.com/code.html>
3. <https://github.com/standard/standard>
4. <https://github.com/Flet/semistandard>
5. <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html>
6. <https://github.com/ericelliott/class-free-javascript-style>

**_UPDATE (2019-07-22)_**

now I use Prettier with the following config:

```javascript
// https://prettier.io/docs/en/options.html

module.exports = {
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true,
  quoteProps: 'as-needed',
  jsxSingleQuote: true,
  trailingComma: 'all',
  bracketSpacing: false,
  jsxBracketSameLine: false,
  arrowParens: 'avoid',
  requirePragma: false,
  insertPragma: true,
  proseWrap: 'always',
};
```

ESLint config is modified accordingly so as not to conflict with Prettier.

## space after function name in function declaration

1. <https://github.com/standard/standard/issues/217>

imho it's a very controversial rule but I'll try to stick it for the time being.

## underscore for private properties

1. <https://github.com/airbnb/javascript/issues/1024#issuecomment-242588541>
2. <https://github.com/ericelliott/class-free-javascript-style#22.4>

## semicolon

1. <https://github.com/Flet/semistandard>
2. <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#semicolons>

use semicolons:

- after all single statements (variable/constant declarations, imports/exports,
  etc.)

don't use semicolons:

- after class and function declarations
- after block statements (blocks) (commonly used with control flow statements -
  `if...else`, `for`, etc.)

  1. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/block>

adapted example from <https://facebook.github.io/react/tutorial/tutorial.html>:

```javascript
import React from 'react';

class Board extends React.Component {
  render() {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div>
        <div className='status'>{status}</div>
        <div className='board-row'>
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
      </div>
    );
  }
}
```

## single quotes

1. <https://standardjs.com/>
2. <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#quotes>

prefer single quotes over double ones.

## parentheses around parameters of arrow functions

1. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions>

I use `arrow-parens` ESLint rule:

```yaml
# .eslintrc.yml

arrow-parens:
  - error
  - as-needed
  - requireForBlockBody: true
```

=> parameter is not wrapped only if it's single and if function body is an
instructions block (surrounded by braces).

```javascript
() => 1
v => v + 1
(v, i) => v + 1
```

## checking for null, undefined or empty string

1. <https://stackoverflow.com/a/5515349/3632318>
2. <https://www.sitepoint.com/javascript-truthy-falsy/>

check that variable has a truthy value instead of checking for `null`,
`undefined` or empty string explicitly but only when that variable can be an
empty string:

```javascript
// variable must be declared before making checks
const foo = '';

// good (using `!!foo` to get boolean value is usually redundant)
if (foo) {
  console.log(foo);
} // nothing printed
// bad (more verbose)
if (foo != null && foo.length !== 0) {
  console.log(foo);
} // nothing printed
```

if variable can't be an empty string, prefer checking for `null` and `undefined`
explicitly:

```javascript
const foo = {a: 1};

// good
if (foo != null) {
  console.log('foo');
}
// worse (assumes that foo can be empty string which can be misleading)
if (foo) {
  console.log('foo');
}
```
