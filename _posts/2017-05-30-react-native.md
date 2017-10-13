---
layout: post
title: React Native
date: 2017-05-30 15:54:03 +0300
access: public
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

## flexbox

- <https://css-tricks.com/snippets/css/a-guide-to-flexbox/>
- <https://facebook.github.io/react-native/docs/layout-props.html#flex>

all RN components implicitly have `display: flex`.

flex items (children) are placed inside flex container (parent).

flex items have `box-sizing: content-box` (not `border-box` - that is
padding and border are not included in component's width and height).

### flex item properties

- `flex`

  - `0` - component is inflexible, sized according to its `width` and `height`
  - `-1` - component is inflexible, sized according to its `width` and `height`,
    can be shrinked to its `minWidth` and `minHeight`
  - `1` - component is flexible, sized proportional to specified flex value
    (can be > 1)

## networking

<https://facebook.github.io/react-native/docs/network.html>

RN provides `Fetch API` that allows to fetch content from arbitrary URL:

```javascript
function getUsers () {
  return fetch('https://test.com/users.json')
    .then(response => response.json())
    .then(responseJson => responseJson.users)
    .catch(error => console.error(error));
}
```

`Fetch` methods (as seen above) return promises.

example can be rewritten using `async`/`await` syntax from ES2017:

```javascript
async function getUsers () {
  try {
    let response = await fetch('https://test.com/users.json');
    let responseJson = await response.json();
    return responseJson.users;
  } catch(error) {
    console.error(error);
  }
}
```

## linking native libraries

<http://facebook.github.io/react-native/docs/linking-libraries-ios.html>

native libraries can be linked:

- automatically

  ```sh
  $ npm install --save <library-with-native-dependencies>
  $ react-native link
  ```

  this method is for npm packages that ship with native libraries.

- manually

  by adding corresponding library (its Xcode project file) to
  your application (see the guide above).

  this method is for independent static libraries that ship with RN -
  they are not included by default so as not to impact binary size.

## updating components

component is updated when:

- its internal state has changed (using `setState`)
- `forceUpdate` is triggered (say, when Redux store is changed)
- props have changed:

  parent component state changes -\>
  it's re-rendered -\>
  current component receives new props

- props in `mapStateToProps` have changed:

  component is connected to Redux store -\>
  Redux store state has changed -\>
  props in `mapStateToProps` have changed
  (they are compared using shallow comparison)
