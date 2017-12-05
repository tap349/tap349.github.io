---
layout: post
title: JavaScript - Style Guide
date: 2017-06-10 18:36:47 +0300
access: public
comments: true
categories: [js]
---

<!-- more -->

1. <https://standardjs.com/>
2. <http://javascript.crockford.com/code.html>
3. <https://github.com/feross/standard>
4. <https://github.com/Flet/semistandard>
5. <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html>
6. <https://github.com/ericelliott/class-free-javascript-style>

## space after function name in function declaration

1. <https://github.com/feross/standard/issues/217>

imho it's a very controversial rule but I'll try to stick it for the time being.

## underscore for private properties

1. <https://github.com/airbnb/javascript/issues/1024#issuecomment-242588541>
2. <https://github.com/ericelliott/class-free-javascript-style#22.4>

## semicolon

1. <https://github.com/Flet/semistandard>
2. <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#semicolons>

use semicolons:

- after all single statements (variable/constant declarations, imports/exports, etc.)

don't use semicolons:

- after class and function declarations
- after [block statements (blocks)](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/block)
  (commonly used with control flow statements - `if...else`, `for`, etc.)

adapted example from <https://facebook.github.io/react/tutorial/tutorial.html>:

```javascript
import React from 'react';

class Board extends React.Component {
  render () {
    const winner = calculateWinner(this.state.squares);
    let status;
    if (winner) {
      status = 'Winner: ' + winner;
    } else {
      status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
      <div>
        <div className="status">{status}</div>
        <div className="board-row">
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

don't wrap single parameter of arrow function in parentheses:

```javascript
v => v + 1
(v, i) => v + 1
```
