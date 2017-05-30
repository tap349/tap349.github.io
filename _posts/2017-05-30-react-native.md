---
layout: post
title: React Native
date: 2017-05-30 15:54:03 +0300
access: public
categories: [react-native]
---

<!-- more -->

## Networking

<https://facebook.github.io/react-native/docs/network.html>

RN provides `Fetch API` that allows to fetch content from arbitrary URL:

```javascript
function getUsers() {
  return fetch('https://test.com/users.json')
    .then(response => response.json())
    .then(responseJson => responseJson.users)
    .catch(error => console.error(error))
}
```

`Fetch` methods (as seen above) return promises.

example can be written using `async`/`await` syntax from ES2017:

```javascript
async function getUsers() {
  try {
    let response = await fetch('https://test.com/users.json');
    let responseJson = await response.json();
    return responseJson.users;
  } catch(error) {
    console.error(error)
  }
}
```
