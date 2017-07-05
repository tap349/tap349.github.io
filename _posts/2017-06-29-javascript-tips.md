---
layout: post
title: JavaScript - Tips
date: 2017-06-29 17:44:10 +0300
access: public
categories: []
---

<!-- more -->

* TOC
{:toc}

## convert array to object keyed by ID for Redux

- <http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html>
- <https://medium.com/dailyjs/rewriting-javascript-converting-an-array-of-objects-to-an-object-ec579cafbfc7>

```javascript
// [{id: 2, name: 'foo'}, {id: 2, name: 'bar'}] ->
//  {2: {id: 1, name: 'foo'}, 2: {id: 2, name: 'bar'}}
const arrayToObject = (array) =>
  array.reduce((obj, item) => {
    obj[item.id] = item;
    return obj;
  }, {})
```

NOTE: original array sorting is lost in resulting object!

if you need to keep sorting opt for `Map` instead:

```javascript
const arrayToMap = (array) =>
  array.reduce((map, item) => {
    map.set(item.id, item);
    return map;
  }, new Map());
```

## merge objects

- using `Object.assign`

  <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#Merging_objects>

  ```javascript
  Object.assign({a: 1, b: 2}, {b: 3}) // {a: 1, b: 3}
  ```

- using spread operator (`...`)

  <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator#Spread_in_object_literals>

  ```javascript
  {...{a: 1, b: 2}, ...{b: 3}} // {a: 1, b: 3}
  ```

  while `Object.assign` modifies target object using spread syntax creates new object.

## pass class prototype methods as arguments

- <https://stackoverflow.com/questions/35814872/es6-class-pass-function-as-parameter>
- <https://stackoverflow.com/questions/35446486/binding-a-function-passed-to-a-component>

if it's necessary to keep current context (say, class method uses
instance properties of current class) there are 2 options:

- pass prototype method and bind it to current context in-place

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

- [RECOMMENDED] use field declaration to define prototype method as arrow function

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

  now `handleResponse` is bound to `Foo` class instance forever
  (it's even impossible to rebind it explicitly using `bind`).
