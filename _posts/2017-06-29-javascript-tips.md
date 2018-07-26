---
layout: post
title: JavaScript - Tips
date: 2017-06-29 17:44:10 +0300
access: public
comments: true
categories: [js]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) merge objects
----------------------

- using `Object.assign`

  1. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign#Merging_objects>

  ```javascript
  Object.assign({a: 1, b: 2}, {b: 3}) // {a: 1, b: 3}
  ```

- [ES6] using spread operator (`...`)

  1. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator#Spread_in_object_literals>

  {% raw %}
  ```javascript
  {...{a: 1, b: 2}, ...{b: 3}} // {a: 1, b: 3}
  ```
  {% endraw %}

  while `Object.assign` modifies target object using spread syntax creates new object.

(how to) pass class prototype methods as arguments
--------------------------------------------------

1. <https://stackoverflow.com/questions/35814872/es6-class-pass-function-as-parameter>
2. <https://stackoverflow.com/questions/35446486/binding-a-function-passed-to-a-component>

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

- *[RECOMMENDED]* use field declaration to define prototype method as arrow function

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

(how to) print form data (multipart/form-data)
----------------------------------------------

```javascript
console.log('formData: ' + JSON.stringify(formData));
```

(how to) post form data (application/x-www-form-urlencoded)
-----------------------------------------------------------

1. <https://github.com/facebook/react-native/issues/3349>
2. <https://stackoverflow.com/a/32445457/3632318>

```javascript
const jsonToFormData = (json) => {
  return Object.entries(json)
    .map(entry => {
      const [key, value] = entry;
      const encodedKey = encodeURIComponent(key);
      const encodedValue = encodeURIComponent(value);

      return `${encodedKey}=${encodedValue}`;
    }, [])
    .join('&');
}

const response = fetch(url, {
  method: 'POST',
  headers: {'Content-Type': 'application/x-www-form-urlencoded'},
  body: jsonToFormData(json)
})
```

[ES6] (how to) get subset of object properties
----------------------------------------------

1. <https://stackoverflow.com/questions/17781472>

```javascript
const object = {a: 5, b: 6, c: 7};
const picked = (({a, c}) => ({a, c}))(object);

console.log(picked); // {a: 5, c: 7}
```

or else remove unnecessary properties (see the next tip).

[ES6] (how to) remove object property
-------------------------------------

1. <https://stackoverflow.com/a/33053362/3632318>

to remove static property:

```javascript
const object = {a: 5, b: 6, c: 7};
const {b, ...rest} = object;

console.log(rest); // {a: 5, c: 7}
```

to remove calculated property (say, ID):

```javascript
const object = {1: 'foo', 2: 'bar', 3: 'baz'};
const id = 2;
const {[id]: _value, ...rest} = object;

console.log(rest); // {1: "foo", 3: "baz"}
```

[ES6] (how to) create empty array of N elements
-----------------------------------------------

1. <https://stackoverflow.com/a/41246860/3632318>

```javascript
[...Array(100)]
```

(how to) create array with the same element repeated multiple times
-------------------------------------------------------------------

1. <https://stackoverflow.com/a/34104348/3632318>

```javascript
> Array(7).fill(false)
< [false, false, false, false, false, false, false]
```

[ES6] (how to) conditionally add property to object
---------------------------------------------------

1. <https://stackoverflow.com/a/40560953/3632318>

```javascript
const foo = 1;
const bar = {
  ...!!foo ? {foo} : null,
  baz: 2,
};
```

NOTE: don't use `...!!foo && {foo}` because RN JS server might
      complain about some performance optimizations if something
      else but `false` or `undefined` is expanded inside object.

[ES6] (how to) remove duplicate values from array
-------------------------------------------------

1. <https://stackoverflow.com/a/41364433/3632318>

```javascript
Array.from(new Set([1, 2, 3, 2, 2, 3, 1]))
// => [1, 2, 3]

// or
[...new Set([1, 2, 3, 2, 2, 3, 1])]
// => [1, 2, 3]
```

NOTE:

- insertion order is preserved
- all duplicate values but the 1st one are removed


(how to) flatten array
----------------------

```javascript
const array = [1, [2, 3], 4];
[].concat(...array);
// => [1, 2, 3, 4]
```

(how to) wrap a long template literal to multiline
--------------------------------------------------

1. <https://stackoverflow.com/a/45153504/3632318>

```jsx
<Checkbox
  checkedHint={
    `foo${gc.NBSP}bar ` +
    `baz${gc.NBSP}qux`
  }
/>
```

using line continuation at the point of newline in the literal
preserves all spaces at the beginning of the next line:

```javascript
const variable =
  `foo${gc.NBSP}bar \
   baz${gc.NBSP}qux`; // leading spaces will be preserved
```

[ES6] (how to) export and import
--------------------------------

1. <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export>
2. <https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/import>

- named export

  ```javascript
  //---------------------------------------------------------
  // export.js
  //---------------------------------------------------------

  export const foo = 1;
  export const bar = 2;

  // same as above:
  const foo = 1;
  const bar = 2;
  export {foo, bar};

  // or mix both variants:
  export const foo = 1;
  const bar = 2;
  export {bar};

  //---------------------------------------------------------
  // import.js
  // named exports are imported as a single object
  //---------------------------------------------------------

  // import selected exports using destructuring
  import {foo, bar} from '.export';
  // import all exports
  import * as baz from '.export';
  ```

- default export

  ```javascript
  //---------------------------------------------------------
  // export.js
  //---------------------------------------------------------

  const foo = 1;
  const bar = 2;
  export default {foo, bar};

  //---------------------------------------------------------
  // import.js
  //---------------------------------------------------------

  import baz from '.export';
  ```

  **NOTE**

  it's not possible to use `var`, `let` or `const` with `export default`:
  if it's still necessary to use default export with, say, a constant,
  declare and export it in a separate statements:

  ```javascript
  // not allowed:
  //export default const foo = 123;
  const foo = 123;
  export default foo;
  ```
