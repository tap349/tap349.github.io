---
layout: post
title: Webpack - Phoenix
date: 2018-05-23 02:31:59 +0300
access: private
comments: true
categories: [webpack, phoenix]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://webpack.js.org/configuration/>
2. <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0>
3. <http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html>

prepare file structure
----------------------

```sh
$ mkdir -p assets/{css,js}
$ touch assets/css/app.scss
$ touch assets/js/app.js
$ rm -rf priv/static
```

remove Brunch
-------------

I created Phoenix project with `--no-brunch` option so nothing to do here.

add Webpack
-----------

```sh
$ cd assets
$ yarn add webpack --dev
```

update .gitignore
-----------------

```diff
  # .gitignore

+ /assets/node_modules
+ yarn-debug.log
+ yarn-error.log
```

add package scripts
-------------------

> <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0>
>
> Phoenix hooks into the scripts to compile assets, so we need to change
> the commands.

```diff
  // assets/package.json

  {
+   "scripts": {
+     "start": "yarn run watch",
+     "watch": "webpack --mode development --watch",
+     "deploy": "webpack --mode production"
+   },
  }
```

create skeleton Webpack config
------------------------------

1. <http://phoenixframework.org/blog/static-assets>

```javascript
// assets/webpack.config.js

const path = require('path');

module.exports = {
  entry: './js/app.js',
  output: {
    path: path.resolve(__dirname, '../priv/static'),
    filename: 'js/app.js',
  },
  module: {
    rules: [],
    plugins: [],
  },
};
```

add Babel loader
----------------

> <https://www.npmjs.com/package/babel-preset-env>
>
> Without any configuration options, babel-preset-env behaves exactly the
> same as babel-preset-latest (or babel-preset-es2015, babel-preset-es2016,
> and babel-preset-es2017 together).

- add npm packages

  ```sh
  $ cd assets
  $ yarn add babel-core babel-loader babel-preset-env --dev
  ```

- create Babel config

  ```javascript
  // assets/.babelrc

  {
    'presets': [
      ['env', {'modules': false}],
    ],
  }
  ```

  it's possible to specify Babel options right inside Webpack config:

  ```diff
    // assets/webpack.config.js

    {
      test: /\.js$/,
      loader: 'babel-loader',
  +   options: {
  +     presets: ['env', {'modules': false}],
  +   },
    },
  ```

- update Webpack config

  ```javascript
  // assets/webpack.config.js

  module.exports = {
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          loader: 'babel-loader',
        },
      ],
    },
  };
  ```

  these are equivalent ways to specify loader:

  ```javascript
  // 1 (single loader, with options)
  {
    test: /\.js$/,
    loader: 'babel-loader',
    options: {},
  }

  // 2 (single loader, with options)
  {
    test: /\.js$/,
    use: {
      loader: 'babel-loader',
      options: {},
    }
  }

  // 3 (multiple loaders, without options)
  {
    test: /\.js$/,
    use: ['babel-loader'],
  }

  // 4 (multiple loaders, with options)
  {
    test: /\.js$/,
    use: [
      {
        loader: 'babel-loader',
        options: {},
      }
    ]
  }
  ```

add Sass loader
---------------

1. <https://github.com/webpack-contrib/sass-loader>
2. <https://github.com/webpack-contrib/mini-css-extract-plugin>

> <https://www.valentinog.com/blog/webpack-4-tutorial/#webpack_4_extracting_CSS_to_a_file>
>
> webpack doesn’t know to extract CSS to a file.
> In the past it was a job for extract-text-webpack-plugin.
> Unfortunately said plugin does not play well with webpack 4.
> mini-css-extract-plugin is here to overcome those issues.
> NOTE: Make sure to update webpack to version 4.2.0. Otherwise
> mini-css-extract-plugin won’t work!

- add npm packages

  ```sh
  $ cd assets
  $ yarn add style-loader css-loader sass-loader node-sass --dev
  $ yarn add mini-css-extract-plugin --dev
  $ yarn add optimize-css-assets-webpack-plugin uglifyjs-webpack-plugin --dev
  ```

- update Webpack config

  > <https://medium.com/@ahmedelgabri/a-chunk-is-an-on-demand-loaded-js-file-https-webpack-js-org-configuration-output-output-4145373573fb>
  >
  > A chunk is an on-demand loaded js file.
  > A bundle is the normal output of your entry point.

  > <https://github.com/webpack-contrib/mini-css-extract-plugin#minimizing-for-production>
  >
  > While webpack 5 is likely to come with a CSS minimizer built-in,
  > with webpack 4 you need to bring your own. To minify the output,
  > use a plugin like optimize-css-assets-webpack-plugin. Setting
  > optimization.minimizer overrides the defaults provided by webpack,
  > so make sure to also specify a JS minimizer

  ```javascript
  // assets/webpack.config.js

  const MiniCssExtractPlugin = require('mini-css-extract-plugin');
  const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

  const devMode = process.env.NODE_ENV !== 'production';

  module.exports = {
    module: {
      rules: [
        {
          test: /\.(css|scss)$/,
          use: [
            devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
            'css-loader',
            'sass-loader',
          ],
        },
      ],
    },
    plugins: [
      new MiniCssExtractPlugin({
        // I guess this filename is relative to output.path
        filename: devMode ? 'css/[name].css' : 'css/[name].[hash].css',
      }),
    ],
    optimization: {
      minimizer: [
        new UglifyJsPlugin({cache: true, parallel: true, sourceMap: true}),
        new OptimizeCSSAssetsPlugin({}),
      ],
    },
  };
  ```
