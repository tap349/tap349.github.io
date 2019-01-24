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
$ npm install node-sass sass-loader --save-dev
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

milligram
---------

```sh
$ cd assets
$ npm install milligram
```

```scss
// assets/css/app.scss

@import '~milligram/dist/milligram.min.css';
```
