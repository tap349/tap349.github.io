---
layout: post
title: JavaScript
date: 2017-05-30 16:20:16 +0300
access: public
categories: [js]
---

<!-- more -->

## functions

- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions>
- <https://stackoverflow.com/a/155655/3632318>

> Every function in JavaScript is a Function object.

methods are functions associated with an object via its properties which are
implicitly passed that object when called (class instance is an object too):

```javascript
var obj = {
  foo: function() {}
};
```

is equivalent to this using ES6 shorthand syntax:

```javascript
var obj = {
  foo() {}
};
```

it's possible to use only shorthand syntax inside classes.

that is why always use `this` to get current object property that might happen
to be function object and then use parentheses to call that function object.

### constructor function

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new#Description>

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

## template literals

<https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals>

template literals (formerly known as 'template strings') are
string literals which allow multi-line strings and string interpolation:

```javascript
`text line 1
 text line 2`

`text ${expression} text`
```
