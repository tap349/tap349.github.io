---
layout: post
title: JavaScript - ES6+
date: 2017-06-02 23:55:07 +0300
access: public
categories: [js, es6, es7, es8]
---

<!-- more -->

### promise

- <https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-promise-27fc71e77261>
- <https://learn.javascript.ru/promise>
- <https://stackoverflow.com/a/35282921/3632318>
- <https://stackoverflow.com/a/30741722/3632318>

the result of a promise chain is always a promise - either resolved or
rejected one. this is what allows to chain promises endlessly.

resolved or rejected values (i.e. resolved or rejected promise values) will be
passed to corresponding callback functions attached to `then()`.

### shorthand property names

- <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer>
- <http://www.benmvp.com/learning-es6-enhanced-object-literals/>

```javascript
> var a = 'foo', b = 42, c = {};
> var o = {a, b, c};
> o.a;
"foo"
```

### arrow function

<https://stackoverflow.com/questions/33308121/can-you-bind-arrow-functions>

it's impossible to rebind arrow function - just use normal function if
you need to bind it to another context later.

### field declarations

<https://github.com/tc39/proposal-class-fields#field-declarations>

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

### pass class prototype methods as arguments

- <https://stackoverflow.com/questions/35814872/es6-class-pass-function-as-parameter>
- <https://stackoverflow.com/questions/35446486/binding-a-function-passed-to-a-component>

if it's necessary to keep current context (say, class method uses
instance properties of current class) there are 2 options:

- pass prototype method and bind it to current context in-place:

  ```javascript
  class Foo {
    constructor (bar) {
      this.bar = 123;
    }

    foo (userId) {
      Api.get(userId, this.handleResponse.bind(this));
    }

    handleResponse (response) {
      return response.baz + this.bar;
    }
  }
  ```

  also it's possible to do it in constructor once and for all
  (I haven't tested this solution):

  ```javascript
  class Foo {
    constructor (bar) {
      this.bar = 123;
      this.handleResponse = this.handleResponse.bind(this);
    }

    foo (userId) {
      Api.get(userId, this.handleResponse);
    }

    handleResponse (response) {
      return response.baz + this.bar;
    }
  }
  ```

- use field declaration to define prototype method as arrow function:

  ```javascript
  class Foo {
    constructor (bar) {
      this.bar = 123;
    }

    foo (userId) {
      Api.get(userId, this.handleResponse);
    }

    handleResponse = (response) => {
      return response.baz + this.bar;
    }
  }
  ```

  now `handleResponse` is bound to `Foo` class instance forever.
