---
layout: post
title: JavaScript
date: 2017-05-30 16:20:16 +0300
access: public
categories: [js]
---

<!-- more -->

## semicolon

<https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#semicolons>

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

## single vs. double quotes

<https://basarat.gitbooks.io/typescript/content/docs/styleguide/styleguide.html#quotes>

prefer single quotes.

## falsy values

<https://stackoverflow.com/a/5515349/3632318>

- `null`
- `undefined`
- `NaN`
- empty string ("")
- 0
- `false`

## switch statement

- <https://stackoverflow.com/a/6612676/3632318>
- <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/switch>

switch statement doesn't return value - it's used for side effects only.

it's possible to use either `break` or `return` to terminate each case clause.

using `break` is optional but if it's omitted program continues execution:
it jumps to the next case clause and checks whether it's matched against
`switch` expression and so on.

`return` statement returns result from function - it's okay to use it if there
is nothing else after `switch` statement.

## comparison operators

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators>

- equality (`==`)

  perform type conversion (if operands have different types) before applying
  strict comparison, objects are equal if they have the same references.

- identity/strict equality (`===`)

  returns true if operands are strictly equal without type conversion,
  objects are equal if they have the same references.
