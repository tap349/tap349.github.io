---
layout: post
title: JavaScript - Style Guide
date: 2017-06-10 18:36:47 +0300
access: public
comments: true
categories: [js]
---

<!-- more -->

- <https://standardjs.com/>
- <http://javascript.crockford.com/code.html>
- <https://github.com/feross/standard>
- <https://github.com/Flet/semistandard>
- <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html>
- <https://github.com/ericelliott/class-free-javascript-style>

## space after function name in function declaration

<https://github.com/feross/standard/issues/217>

imho it's a very controversial rule but I'll try to stick it for the time being.

## underscore for private properties

- <https://github.com/airbnb/javascript/issues/1024#issuecomment-242588541>
- <https://github.com/ericelliott/class-free-javascript-style#22.4>

## semicolon

- <https://github.com/Flet/semistandard>
- <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#semicolons>

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

- <https://standardjs.com/>
- <https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#quotes>

prefer single quotes over double ones.
