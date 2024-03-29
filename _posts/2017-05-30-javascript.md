---
layout: post
title: JavaScript
date: 2017-05-30 16:20:16 +0300
access: public
comments: true
categories: [js, es6, es7, esnext]
---

<!-- more -->

* TOC
{:toc}
<hr>

functions
---------

1. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Method_definitions>
2. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions>
3. <https://stackoverflow.com/a/155655/3632318>

> Every function in JavaScript is a Function object.

methods are functions associated with an object via its properties which are
implicitly passed that object when called (class instance is an object too):

```javascript
var obj = {
  foo: function () {}
};
```

is equivalent to this using ES6 shorthand syntax:

```javascript
var obj = {
  foo () {}
};
```

it's possible to use only shorthand syntax inside classes.

that is why always use `this` to get current object property that might happen
to be function object and then use parentheses to call that function object.

### constructor function

1. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new#Description>

### functions inside object

1. <https://stackoverflow.com/a/48645842/3632318>

ES5:

```javascript
var myObj = {
  myMethod: function myMethod (params) {
    // ...do something here
  },
};
```

ES6 (the same as above):

```javascript
var myObj = {
  myMethod (params) {
    // ...do something here
  },
};
```

properties
----------

1. <http://diegobarahona.com/javascript/es6/2015/01/05/understanding-es6-classes/>
2. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes#Boxing_with_prototype_and_static_methods>

- static properties

  ```javascript
  function Perro () {}
  Perro.definition = "The dog is the man's best friend";
  ```

  in ES6 these are static methods:

  ```javascript
  class Perro {
    static bark () {}
  }
  ```

  in ES5 `bark` would be called with global object as `this`
  (autoboxing) while in ES6 `bark` is bound to class instance:

  ```javascript
  // ES5
  let perro = new Perro();
  let bark = perro.bark;
  bark(); // global object

  // ES6
  let perro = new Perro();
  let bark = perro.bark;
  bark(); // undefined
  ```

- instance properties

  ```javascript
  function Perro (props) {
    this.color = props.color;
  }
  ```

- prototype properties

  ```javascript
  function Perro () {}
  Perro.prototype.bark = function () {};
  Perro.prototype.walk = function () {};
  ```

  in ES6 these are class methods:

  ```javascript
  class Perro {
    bark () {}
  }
  ```

  same considerations regarding autoboxing as for static properties
  apply here as well.

falsy values
------------

1. <https://stackoverflow.com/a/5515349/3632318>

- `null`
- `undefined`
- `NaN`
- empty string ("")
- 0
- `false`

switch statements
-----------------

1. <https://stackoverflow.com/a/6612676/3632318>
2. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/switch>

switch statement doesn't return value - it's used for side effects only.

it's possible to use either `break` or `return` to terminate each case clause.

using `break` is optional but if it's omitted program continues execution:
it jumps to the next case clause and checks whether it's matched against
`switch` expression and so on.

`return` statement returns result from function - it's okay to use it if there
is nothing else after `switch` statement.

comparison operators
--------------------

1. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators>
2. <https://stackoverflow.com/a/15992131/3632318>

- equality (`==`)

  perform type conversion (if operands have different types) before applying
  strict comparison, objects are equal if they have the same references.

- identity/strict equality (`===`)

  returns true if operands are strictly equal without type conversion,
  objects are equal if they have the same references.

as a rule always prefer strict equality operators except for the case when
it's necessary to test for `null` or `undefined`:

```javascript
// tests both for null and undefined
if (variable == null) {
  ...
}
```

[ES6] template literals
-----------------------

1. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals>

template literals (formerly known as 'template strings') are
string literals which allow multiline strings and string interpolation:

```javascript
`text line 1
text line 2`

`text ${expression} text`
```

[ES6] promises
--------------

1. <https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261>
2. <https://learn.javascript.ru/promise>
3. <https://stackoverflow.com/a/35282921/3632318>
4. <https://stackoverflow.com/a/30741722/3632318>

the result of a promise chain is always a promise - either resolved or
rejected one. this is what allows to chain promises endlessly.

resolved or rejected values (i.e. resolved or rejected promise values) will be
passed to corresponding callback functions attached to `then()`.

### returning promise from handler functions

<https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then>:

> The handler function (onFulfilled or onRejected) gets then called asynchronously
> (as soon as the stack is empty). After the invocation of the handler function,
> if the handler function:
>
> - returns a value, the promise returned by then gets resolved with the returned value as its value;
> - throws an error, the promise returned by then gets rejected with the thrown error as its value;
> - returns an already resolved promise, the promise returned by then gets resolved with that promise's value as its value;
> - returns an already rejected promise, the promise returned by then gets rejected with that promise's value as its value.

[ES6] shorthand property names
------------------------------

1. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer>
2. <http://www.benmvp.com/learning-es6-enhanced-object-literals/>

```javascript
> var a = 'foo', b = 42, c = {};
> var o = {a, b, c};
> o.a;
"foo"
```

[ES6] arrow functions
---------------------

1. <https://stackoverflow.com/questions/33308121/can-you-bind-arrow-functions>

it's impossible to rebind arrow function - just use normal function if
you need to bind it to another context later.

[ESNext] field declarations
---------------------------

1. <https://github.com/tc39/proposal-class-fields#field-declarations>

```javascript
class Counter extends HTMLElement {
  x = 0;
}
```

is equivalent to:

```javascript
class Counter extends HTMLElement {
  constructor () {
    super();
    this.x = 0;
  }
}
```
