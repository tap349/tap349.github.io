---
layout: post
title: React Native
date: 2017-05-30 15:54:03 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

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

_app/components/login/PhonePage.js_:

```javascript
export default class PhonePage extends Component {
  // ...
}
```

_app/components/login/index.js_:

```javascript
import PhonePage from './PhonePage';

export default {
  PhonePage,
};
```

_app/App.js_:

```javascript
import Login from './components/login';

export default class App extends Component {
  // ...
  case 'login_phone_page':
    return <Login.PhonePage navigator={navigator} />;
  // ...
}
```
