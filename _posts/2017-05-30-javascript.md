---
layout: post
title: JavaScript
date: 2017-05-30 16:20:16 +0300
access: public
categories: [js]
---

<!-- more -->

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

using `break` is optional but if it's omitted program continues exepction:
jumps to the next case clause and check whether it's matched against `switch`
expression and so on.

`return` statement returns result from function - it's okay to use it if there
is nothing else after `switch` statement.

## comparison operators

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators>

- equality (`==`)

  type conversion (if operands have different types) => strict comparison,
  objects are equal if they have the same references.

- identity/strict equality (`===`)

  returns true if operands are strictly equal without type conversion,
  objects are equal if they have the same references.

## ES6

### Promise

<https://learn.javascript.ru/promise>

### shorthand property names

- <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer>
- <http://www.benmvp.com/learning-es6-enhanced-object-literals/>

```javascript
> var a = 'foo', b = 42, c = {};
> var o = {a, b, c};
> o.a
"foo"
```
