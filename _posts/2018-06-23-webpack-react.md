---
layout: post
title: Webpack - React
date: 2018-06-23 16:37:46 +0300
access: public
comments: true
categories: [webpack, react]
---

this post describes how to set up React in a project using Webpack -
instructions are geared towards Phoenix project but it must be fairly
easy to adapt them to Rails project using Webpacker.

<!-- more -->

* TOC
{:toc}
<hr>

1. <http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html#configure-react>

install npm packages
--------------------

```sh
$ cd assets
$ yarn add react react-dom prop-types
$ yarn add babel-preset-react --dev
```

### Webpacker

it's possible to run `webpacker:install:react` Webpacker generator to
install required npm packages, add `react` preset to Babel config and
create a sample React component _app/assets/packs/hello_react.jsx_ but
generally it's better to do everything manually.

add `react` preset to Babel config
----------------------------------

```diff
  // assets/.babelrc

  "presets": [
    // ...
+   "react"
  ],
```

don't add `@babel/preset-react` package - it requires Babel 7 which is still
in beta (and Webpacker, say, depends on Babel 6).

add `jsx` extension to be resolved automatically
------------------------------------------------

```javascript
// assets/webpack.config.js

resolve: {
  // ...
  extensions: ['.js', '.jsx', /* ... */],
},
```

### Webpacker

`jsx` extension is added in Webpacker config.

update _app.js_ to load page-specific JS files recursively
----------------------------------------------------------

1. <https://webpack.js.org/guides/dependency-management/#context-module-api>

page-specific JS file is JS file used to load JS for specific page
(all pages should have unique body classes like `p-user-new` where
`p` is a page prefix, `user` is controller name and `new` is action
name).

load all JS files in _assets/js/pages/_ recursively:

```javascript
// assets/js/app.js

const r = require.context('./pages', true, /\.js$/);
r.keys().forEach(r);
```

add element with `react` ID to corresponding template
-----------------------------------------------------

of course you can choose any other ID name and insert this
element wherever you want inside the template:

```slim
// lib/myapp_web/templates/user/new.html.slime

// it's better to serialize user in view helper
#react data-user=Poison.encode!(@user)
```

React component props are passed via data attributes.

create page-specific JS file
----------------------------

say, JS file for page to create new user:

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

import User from 'js/react/components/User';

document.addEventListener('DOMContentLoaded', () => {
  if (!document.body.classList.contains('p-user-new')) {
    return;
  }

  const node = document.getElementById('react');
  const user = JSON.parse(node.dataset.user);

  ReactDOM.render(<User.NewPage user={user} />, node);
});
```

create React component
----------------------

```javascript
// assets/js/react/components/User/index.js

import NewPage from './NewPage';

export default {
  NewPage,
};
```

```javascript
// assets/js/react/components/User/NewPage.js

import React from 'react';

class NewPage extends React.Component {
  render () {
    return (
      <div>Hello {this.props.user.name}!</div>
    );
  }
}

export default NewPage;
```
