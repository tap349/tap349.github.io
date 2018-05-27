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
$ yarn add webpack webpack-cli --dev
```

you'll be prompted to add `webpack-cli` package when running Webpack
for the first time (unless you do it now).

add license to package.json
---------------------------

```diff
  // assets/package.json

  {
+   "license": "MIT",
    // ...
  }
```

or else Webpack will print `warning package.json: No license field`
on every compilation.

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

> <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#b198>
>
> Phoenix hooks into the scripts to compile assets, so we need to change
> the commands.

```diff
  // assets/package.json

  {
+   "scripts": {
+     "deploy": "webpack --mode production",
+     "watch": "webpack --mode development --watch"
+   },
  }
```

there's no `start` script in _assets/package.json_ of newly generated
Phoenix project (when it's created with Brunch support) though in some
tutorials they add it as `yarn run watch`.

TODO: check if `start` script is required
TODO: check if there are orphaned node processes without `--watch-stdin` option
TODO: check if using `webpack --watch` instead of `webpack-dev-server` is okay
      (HMR works)

create skeleton Webpack config
------------------------------

1. <http://phoenixframework.org/blog/static-assets>

> <https://webpack.js.org/guides/caching/>
>
> A simple way to ensure the browser picks up changed files is by using
> output.filename substitutions. The [hash] substitution can be used to
> include a build-specific hash in the filename, however it's even better
> to use the [chunkhash] substitution which includes a chunk-specific hash
> in the filename.

```javascript
// assets/webpack.config.js

const path = require('path');

module.exports = (_env, argv) => {
  // see notes below
  const devMode = argv.mode !== 'production';

  return {
    entry: 'js/app.js',
    output: {
      path: path.resolve(__dirname, '../priv/static'),
      filename: 'js/app-[chunkhash].js',
    },
    module: {
      rules: [],
    },
    resolve: {
      extensions: [],
    },
    plugins: [],
    optimization: {
      minimizer: []
    },
  };
};
```

### NODE_ENV vs. mode

1. <https://github.com/webpack/webpack/issues/6460#issuecomment-386947990>
2. <https://webpack.js.org/configuration/configuration-types/#exporting-a-function>

in brief: `NODE_ENV` environment variable is not set by Webpack - export
function from your Webpack config and use `argv.mode` to get current mode:

```javascript
// assets/webpack.config.js

module.exports = (_env, argv) => {
  console.log(argv.mode); // => development or production
  return {
    // config
  };
};
```

if you want to use `process.env.NODE_ENV`, set it manually on command line:

```sh
$ NODE_ENV=production webpack --mode=production
```

so it looks like Webpack 4 tries to deprecate using `process.env.NODE_ENV`
in favour of `argv.mode` to fetch current environment inside Webpack config.

add Babel loader
----------------

1. <https://github.com/babel/babel-loader>

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

  module.exports = (_env, argv) => {
    return {
      module: {
        rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
          },
        ],
      },
      resolve: {
        extensions: ['.js'],
      },
    };
  };
  ```

  these are equivalent ways to configure loader:

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

### JS source maps

1. <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#3a9b>

```diff
  // assets/webpack.config.js

  resolve: {
    // ...
  },
+ devtool: devMode ? 'eval' : 'cheap-module-source-map',
```

`cheap-module-source-map` variant is used for production by Webpacker.

add Sass loader
---------------

1. <https://github.com/webpack-contrib/sass-loader>
2. <https://github.com/webpack-contrib/mini-css-extract-plugin>

> <https://www.valentinog.com/blog/webpack-4-tutorial/#webpack_4_extracting_CSS_to_a_file>
>
> webpack doesn’t know how to extract CSS to a file.
> In the past it was a job for extract-text-webpack-plugin.
> Unfortunately said plugin does not play well with webpack 4.
> mini-css-extract-plugin is here to overcome those issues.
>
> NOTE: Make sure to update webpack to version 4.2.0. Otherwise
> mini-css-extract-plugin won’t work!

don't use CSS file (say, _assets/css/app.scss_) as an entry point
(just like in Webpacker only JS files are allowed to be packs which
are entry points under the hood):

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

  > <https://github.com/sass/node-sass#includepaths>
  >
  > **includePaths**
  >
  > An array of paths that LibSass can look in to attempt to resolve
  > your @import declarations.

  ```javascript
  // assets/webpack.config.js

  const MiniCssExtractPlugin = require('mini-css-extract-plugin');
  const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

  module.exports = (_env, argv) => {
    const devMode = argv.mode !== 'production';

    return {
      module: {
        rules: [
          {
            test: /\.(css|sass|scss)$/,
            use: [
              // fallback to style-loader in development
              devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
              'css-loader',
              'sass-loader',
            ],
          },
        ],
      },
      resolve: {
        extensions: ['.css', '.sass', '.scss'],
      },
      plugins: [
        new MiniCssExtractPlugin({
          // this filename is relative to output.path
          //
          // I use [hash] instead of [chunkhash] (like in output.filename)
          // because it's used in basic mini-css-extract-plugin example.
          //
          // https://github.com/webpack-contrib/mini-css-extract-plugin#long-term-caching:
          // use [contenthash] for long term caching.
          filename: devMode ? 'css/app.css' : 'css/app-[hash].css',
          // IDK when chunkFilename is used so I don't set it here
        }),
      ],
      optimization: {
        minimizer: [
          new UglifyJsPlugin({
            cache: true,
            parallel: true,
            sourceMap: true,
          }),
          new OptimizeCSSAssetsPlugin({}),
        ],
      },
    };
  };
  ```

- update entry point file

  > <https://github.com/webpack-contrib/extract-text-webpack-plugin/issues/552#issuecomment-310538191>
  >
  > webpack natively only understands 'js-ish' files and using a 'css-ish' file
  > as an entry point isn't recommended (imho it should even fail), so you will
  > get a dummy bundle per css entry to 'handle' that. require/import the css
  > inside an entrypoint instead and let extract-text-webpack-plugin emit the
  > CSS file. It removes the CSS from the bundle anyways :)

  ```javascript
  // assets/js/app.js

  import '../css/app.scss';
  ```

### CSS source maps

1. <https://github.com/webpack-contrib/mini-css-extract-plugin/issues/141>

- `devtool` Webpack option has no effect on CSS source maps

  most likely it's used to enable JS source maps only.

- `sourceMap` options of Sass and CSS loaders don't work

  > <https://github.com/webpack-contrib/sass-loader#source-maps>
  >
  > To enable CSS source maps, you'll need to pass the sourceMap option
  > to the sass-loader and the css-loader.

  ```javascript
  // assets/webpack.config.js

  {
    test: /\.(css|sass|scss)$/,
    use: [
      devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
      {loader: 'css-loader', options: {sourceMap: true}},
      {loader: 'sass-loader', options: {sourceMap: true}},
    ],
  },
  ```

  =\> no CSS source maps are generated.

solution is to add `map: {inline: false}` CSS processor option (`cssnano`
minifier is used by default) in `OptimizeCSSAssetsPlugin` configuration:

```diff
  // assets/webpack.config.js

  optimization: {
    minimizer: [
      // ...
-     new OptimizeCSSAssetsPlugin({}),
+     new OptimizeCSSAssetsPlugin({
+       cssProcessorOptions: {
+         discardComments: {removeAll: true},
+         map: {inline: false},
+       },
      }),
    ],
  },
```

add manifest
------------

1. <https://github.com/danethurber/webpack-manifest-plugin>

> <https://webpack.js.org/concepts/manifest/>
>
> As the compiler enters, resolves, and maps out your application, it keeps
> detailed notes on all your modules. This collection of data is called the
> "Manifest" and it's what the runtime will use to resolve and load modules
> once they've been bundled and shipped to the browser.

- add npm package

  ```sh
  $ yarn add webpack-manifest-plugin --dev
  ```

- update Webpack config

  ```javascript
  // assets/webpack.config.js

  const ManifestPlugin = require('webpack-manifest-plugin');

  module.exports = (_env, argv) => {
    return {
      plugins: [
        new ManifestPlugin(),
      ],
    };
  };
  ```

  > <https://github.com/danethurber/webpack-manifest-plugin>
  >
  > This will generate a manifest.json file in your root output directory
  > with a mapping of all source file names to their corresponding output
  > file.

  sample _manifest.json_:

  ```json
  {
    "main.css": "css/app-c21559a82e348ab1c9df.css",
    "main.js": "js/app-1fa165a45a93fa8fd917.js"
  }
  ```

copy static assets
------------------

TODO: CopyWebpackPlugin? (http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html)
      what about adding hashes? file-loader does it?
