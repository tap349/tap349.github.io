---
layout: post
title: JavaScript - Tips
date: 2017-06-29 17:44:10 +0300
access: public
categories: []
---

<!-- more -->

## convert array to object for Redux

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
