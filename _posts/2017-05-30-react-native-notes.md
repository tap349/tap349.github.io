---
layout: post
title: React Native - Notes
date: 2017-05-30 15:54:03 +0300
access: public
comments: true
categories: [react-native]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## networking

1. <https://facebook.github.io/react-native/docs/network.html>

RN provides `Fetch API` that allows to fetch content from arbitrary URL:

```javascript
function getUsers() {
  return fetch('https://test.com/users.json')
    .then(response => response.json())
    .then(responseJson => responseJson.users)
    .catch(error => console.error(error));
}
```

example can be rewritten using `async`/`await` syntax from ES2017:

```javascript
async function getUsers() {
  try {
    let response = await fetch('https://test.com/users.json');
    let responseJson = await response.json();
    return responseJson.users;
  } catch (error) {
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

  by adding corresponding library (its Xcode project file) to your application
  (see the guide above).

  this method is for independent static libraries that ship with RN - they are
  not included by default so as not to impact binary size.

## component naming conventions and file structure

1. <https://github.com/react-toolbox/react-toolbox/issues/98>

```
.
├── app
│   ├── components
│   │   ├── login
│   │   │   ├── Layout.js
│   │   │   └── PhonePage.js
```

```javascript
// app/components/login/PhonePage.js

export default class PhonePage extends Component {
  // ...
}
```

```javascript
// app/components/login/index.js

import PhonePage from './PhonePage';

export default {
  PhonePage,
};
```

```javascript
// app/App.js

import Login from './components/login';

export default class App extends Component {
  // ...
  case 'login_phone_page':
    return <Login.PhonePage navigator={navigator} />;
  // ...
}
```

## store.dispatch() is synchronous

> <https://stackoverflow.com/a/43188641/3632318>
>
> dispatch is synchronous by default, so, as long as you don't have async calls
> in the action creator, they execute in the order they were declared.

> <https://stackoverflow.com/questions/43188378/guaranteed-dispatch-sequence#comment73457568_43188641>
>
> Yes, the original store.dispatch() function is 100% synchronous, and when it
> returns the state has been updated. Any asynchronicity happens if middleware
> intercept the action and do something different.
