---
layout: post
title: JavaScript - ES6+
date: 2017-06-02 23:55:07 +0300
access: public
categories: [js, es6, es7, es8]
---

<!-- more -->

### promise

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

### arrow function

<https://stackoverflow.com/questions/33308121/can-you-bind-arrow-functions>

it's impossible to rebind arrow function - just use normal function if
you need to bind to another context later.

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
  constructor() {
    super();
    this.x = 0;
  }
}
```

### pass class instance method as argument

<https://stackoverflow.com/questions/35814872/es6-class-pass-function-as-parameter>

if it's necessary to keep current context (say, instance method uses instance
properties of current class) there are 2 options:

- pass instance method and bind it to current context in-place:

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

- use field declaration to define instance method as arrow function:

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

  now `handleResponse` is bound to `Foo` instance forever.
