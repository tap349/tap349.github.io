---
layout: post
title: Webpack - Phoenix 1.4
date: 2019-01-24 14:05:06 +0300
access: public
comments: true
categories: [webpack, phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

SCSS
----

```sh
$ cd assets
$ npm install --save-dev node-sass sass-loader
```

```sh
$ mv assets/css/app.css assets/css/app.scss
```

```diff
  // assets/js/app.js

- import css from "../css/app.css"
+ import css from "../css/app.scss"
```

```diff
  // assets/webpack.config.js

  module.exports = (env, options) => ({
    module: {
      rules: [
-       {
-         test: /\.css$/,
-         use: [MiniCssExtractPlugin.loader, 'css-loader']
-       }
+       {
+         test: /\.(css|scss)$/,
+         use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']
+       }
      ]
    }
  });
```

npm packages
------------

IMO there is no need to import minimized versions (_*.min.css_) in
_assets/css/app.scss_ - CSS files will be minimized by Webpack with
`optimize-css-assets-webpack-plugin`.

### milligram

```sh
$ cd assets
$ npm install milligram
```

```scss
// assets/css/app.scss

@import '~milligram/dist/milligram.css';
```

### toastr

```sh
$ cd assets
$ npm install toastr
```

```javascript
// assets/js/app.js

import Toastr from "toastr";

Toastr.success("Hello World!");
```

```scss
// assets/css/app.scss

@import '~toastr/build/toastr.css';
```
