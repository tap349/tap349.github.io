---
layout: post
title: JavaScript - Tips
date: 2017-06-29 17:44:10 +0300
access: public
categories: []
---

<!-- more -->

## convert array to object for Redux

- <https://medium.com/dailyjs/rewriting-javascript-converting-an-array-of-objects-to-an-object-ec579cafbfc7>
- <http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html>

```javascript
// [{id: 2, name: 'foo'}, {id: 2, name: 'bar'}] ->
//  {2: {id: 1, name: 'foo'}, 2: {id: 2, name: 'bar'}}
const arrayToObject = (array) =>
  array.reduce((obj, item) => {
    obj[item.id] = item;
    return obj;
  }, {})
```
