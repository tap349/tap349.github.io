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
