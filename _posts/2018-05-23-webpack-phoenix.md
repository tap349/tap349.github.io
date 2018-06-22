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

you'll be prompted to add `webpack-cli` package when running Webpack for the
first time (unless you do it now).

add license to package.json
---------------------------

```diff
  // assets/package.json

  {
+   "license": "MIT",
    // ...
  }
```

or else Webpack will print `warning package.json: No license field` on every
compilation.

update .gitignore
-----------------

```conf
# .gitignore

/assets/node_modules
yarn-debug.log
yarn-error.log
```

add package scripts
-------------------

> <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#b198>
>
> Phoenix hooks into the scripts to compile assets, so we need to change the
> commands.

```diff
  // assets/package.json

  {
+   "scripts": {
+     "deploy": "webpack --mode production"
+   },
  }
```

NOTE: `watch` script will be added later.

create skeleton Webpack config
------------------------------

1. <http://phoenixframework.org/blog/static-assets>

```javascript
// assets/webpack.config.js

const path = require('path');

module.exports = (_env, argv) => {
  // see notes below
  const devMode = argv.mode !== 'production';

  return {
    // entry point chunk name is 'main' by default
    entry: {app: 'js/app.js'},
    output: {
      path: path.resolve(__dirname, '../priv/static'),
      // include [chunkhash] when not using Phoenix:
      // filename: 'js/[name]-[chunkhash].js'
      filename: 'js/[name].js',
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

> <https://webpack.js.org/configuration/entry-context/#naming>
>
> If a string or array of strings is passed, the chunk is named `main`.
> If an object is passed, each key is the name of a chunk, and the value
> describes the entrypoint for the chunk.

### [hash] substitutions in filenames

1. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Digest.html>

> <https://webpack.js.org/guides/caching/>
>
> A simple way to ensure the browser picks up changed files is by using
> output.filename substitutions. The [hash] substitution can be used to
> include a build-specific hash in the filename, however it's even better
> to use the [chunkhash] substitution which includes a chunk-specific hash
> in the filename.

but when using Phoenix there is no need to include build-specific hash in
filenames - `phx.digest` Mix task creates digested and compressed versions
of all static files in _priv/static/_ (this is where Phoenix searches for
precompiled assets by default) along with a cache static manifest.

don't use `[hash]` substitutions in both environments:

- in development they are prohibited when using Webpack development server
- in production they are unnecessary because `phx.digest` Mix task digests
  static files in _priv/static/_ by itself (or else digested static files
  will have 2 hash suffixes)

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

  ```javascript
  // assets/webpack.config.js

  {
    test: /\.js$/,
    loader: 'babel-loader',
    options: {
      presets: ['env', {'modules': false}],
    },
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

```javascript
// assets/webpack.config.js

resolve: { /* ... */ },
devtool: devMode ? 'eval' : 'cheap-module-source-map',
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

don't use CSS file (say, _assets/css/app.scss_) as an entry point (just
like in Webpacker only JS files are allowed to be packs which are entry
points under the hood):

- add npm packages

  ```sh
  $ cd assets
  $ yarn add style-loader css-loader sass-loader node-sass --dev
  $ yarn add mini-css-extract-plugin --dev
  $ yarn add optimize-css-assets-webpack-plugin uglifyjs-webpack-plugin --dev
  ```

- update Webpack config

  > <https://stackoverflow.com/questions/42523436>
  >
  > If you don't want all of your code be put into a single huge bundle
  > you will split it into multiple bundles which are called chunks in
  > webpack terminology.
  >
  > Typically, chunks directly correspond with the output bundles however,
  > there are some configurations that don't yield a one-to-one relationship.

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
        // MiniCssExtractPlugin is not used in dev mode =>
        // no need to set different filename for dev mode though
        // it's not required when not including hash in filename
        //
        // filename is relative to output.path
        // IDK when and how chunkFilename is used so I don't set it here
        //
        // include [hash] when not using Phoenix
        // (use [contenthash] instead of [hash] for long term caching):
        // new MiniCssExtractPlugin({filename: 'css/[name]-[hash].css'})
        new MiniCssExtractPlugin({filename: 'css/[name].css'}),
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

### style-loader

1. <https://www.hostinger.com/tutorials/difference-between-inline-external-and-internal-css>

> <https://github.com/webpack-contrib/style-loader>
>
> Style Loader
> Adds CSS to the DOM by injecting a <style> tag.

`style-loader`:

- doesn't extract CSS into CSS output bundles

  this is a job of `MiniCssExtractPlugin` - to extract CSS into CSS output
  bundles (separate files, one CSS file per JS file containing CSS).

- creates CSS module for each imported (inside entry point) CSS file

  these CSS modules are defined in JS code generated by `style-loader`
  inside JS output bundle and contain CSS from corresponding CSS files.

- generates JS code to inline all CSS with `<style>` tag

  JS code inserts `<style>` tag into `<head>` element of the DOM and puts
  CSS from all CSS modules inside this tag (we get so called internal CSS
  as a result).

  this JS code is evaluated in browser when JS output bundle is loaded.

=> separate CSS output bundles are never generated during asset compilation
when using `style-loader`.

### CSS source maps

1. <https://github.com/webpack-contrib/mini-css-extract-plugin/issues/141>

- `style-loader`

  `style-loader` doesn't support generating CSS source maps at all - use
  `MiniCssExtractPlugin` instead (actually `style-loader` does allow to
  *enable* source maps but IDK what this means exactly).

  but `MiniCssExtractPlugin` doesn't support HMR yet → use `style-loader`
  in development mode.

- `devtool` Webpack option

  `devtool` option has effect on generating CSS source maps in development
  mode only - set it to any `source-map`-like value to generate CSS source
  map in development mode.

  CSS source maps are always generated in production mode (provided
  `map: {inline: false}` CSS processor option is passed - see below).

- `sourceMap` options of Sass and CSS loaders

  `sourceMap` options of both loaders seem to have no effect on generating
  CSS source maps.

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

  => nothing changes if `sourceMap` option is set to `false` or not set at all.

- `map: {inline: false}` CSS processor option

  CSS source maps are never generated unless `map: {inline: false}` CSS
  processor (`cssnano` by default) option is added to configuration of
  `OptimizeCSSAssetsPlugin`:

  ```javascript
  // assets/webpack.config.js

  optimization: {
    minimizer: [
      // ...
      new OptimizeCSSAssetsPlugin({
        cssProcessorOptions: {
          discardComments: {removeAll: true},
          map: {inline: false},
        },
      }),
    ],
  },
  ```

**summary**

to generate CSS source maps in production mode:

- use `MiniCssExtractPlugin` (not `style-loader`)
- add `map: {inline: false}` CSS processor option to configuration of
  `OptimizeCSSAssetsPlugin`

to generate CSS source maps in development mode:

- same requirements as for production mode
- set `devtool` to `source-map`-like value

add manifest
------------

1. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Digest.html>

in Phoenix cache static manifest is generated by `phx.digest` Mix task -
there is no need to generate it separately using Webpack plugin.

> <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
>
> `:cache_static_manifest` - a path to a json manifest file that
> contains static files and their digested versions. This is
> typically set to "priv/static/cache_manifest.json" which is the
> file automatically generated by `mix phx.digest`.

cache static manifest is saved to _priv/static/cache_manifest.json_ by default
but its location can be configured in _config/prod.exs_:

```elixir
config :my_app, MyAppWeb.Endpoint,
  # ...
  cache_static_manifest: "priv/static/cache_manifest.json"
```

### webpack-manifest-plugin (for reference only)

1. <https://github.com/danethurber/webpack-manifest-plugin>

> <https://webpack.js.org/concepts/manifest/>
>
> As the compiler enters, resolves, and maps out your application, it keeps
> detailed notes on all your modules. This collection of data is called the
> "Manifest" and it's what the runtime will use to resolve and load modules
> once they've been bundled and shipped to the browser.

- add npm package

  ```sh
  $ cd assets
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
  > files.

  sample _manifest.json_:

  ```json
  {
    "app.css": "css/app-b6889e3643d039f89700.css",
    "app.js": "js/app-14822afe1edfebfb58bb.js"
  }
  ```

#### source file name of CSS source map

`webpack-manifest-plugin` uses chunk path (filename) as a source file name when
chunk name is empty:

```javascript
// https://github.com/danethurber/webpack-manifest-plugin/blob/v2.0.3/lib/plugin.js#L67

var name = chunk.name ? chunk.name : null;

if (name) {
  name = name + '.' + this.getFileType(path);
} else {
  // For nameless chunks, just map the files directly.
  name = path;
}
```

this is the case with CSS source map (note chunk name of CSS source map):

```
$ cd assets
$ yarn run deploy
...
                                Asset       Size  Chunks             Chunk Names
    css/app-a8792a1a9c04c2709540.css   74 bytes       0  [emitted]  app
      js/app-ad744cb644e36e0d32b4.js  644 bytes       0  [emitted]  app
css/app-a8792a1a9c04c2709540.css.map  190 bytes          [emitted]
  js/app-ad744cb644e36e0d32b4.js.map   2.18 KiB       0  [emitted]  app
                       manifest.json  241 bytes          [emitted]
```

that is why CSS source map has a source file name in _manifest.json_ that
looks different compared to other source file names (in which chunk names
are used):

```json
{
  "app.css": "css/app-b6889e3643d039f89700.css",
  "app.js": "js/app-14822afe1edfebfb58bb.js",
  "app.js.map": "js/app-14822afe1edfebfb58bb.js.map",
  "css/app-b6889e3643d039f89700.css.map": "css/app-b6889e3643d039f89700.css.map"
}
```

however CSS source map chunk has a name when Webpack is run in development mode
(requirements for generating CSS source maps in development mode must be met of
course):

```
$ cd assets
$ yarn run deploy
...
                               Asset       Size  Chunks             Chunk Names
    css/app-b11624419d532ffa8ee3.css   81 bytes     app  [emitted]  app
      js/app-275943b67b97d5ad0a35.js    3.4 KiB     app  [emitted]  app
css/app-b11624419d532ffa8ee3.css.map  184 bytes     app  [emitted]  app
  js/app-275943b67b97d5ad0a35.js.map   2.69 KiB     app  [emitted]  app
                       manifest.json  208 bytes          [emitted]
```

it looks like `MiniCssExtractPlugin` is responsible for generating CSS
source maps - it makes sense to search for the answer in its source code.

copy static assets
------------------

1. <http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html>
2. <https://github.com/webpack/webpack/issues/86#issuecomment-350365453>

```sh
$ cd assets
$ yarn add copy-webpack-plugin --dev
$ mkdir -p static/images/
```

```diff
  // assets/webpack.config.js

  const MiniCssExtractPlugin = require('mini-css-extract-plugin');
+ const CopyWebpackPlugin = require('copy-webpack-plugin');
  const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
  const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');

  module.exports = (_env, argv) => {
    const devMode = argv.mode !== 'production';

    return {
    // ...
      plugins: [
        new MiniCssExtractPlugin({filename: 'css/[name].css'}),
+       // contents of assets/static/ is copied to output.path
+       // (priv/static/) by default
+       new CopyWebpackPlugin(['static/']),
      ],
    };
  };
```

`'static/'` is a shorthand for `{from: 'static/'}` when default destination
(`output.path`) is used:

> <https://github.com/webpack-contrib/copy-webpack-plugin>
>
> A simple pattern looks like this
>
> { from: 'source', to: 'dest' }
>
> Or, in case of just a `from` with the default destination, you can
> also use a {String} as shorthand instead of an {Object}: 'source'

just like in case of CSS, don't include `[hash]` substitutions in filenames
of static assets.

also it's possible to add image extensions to `resolve.extensions` array if
you're going to reference image files without specifying their extensions:

```javascript
// assets/webpack.config.js

resolve: {
  extensions: [/* ... */, '.jpg', '.jpeg', '.png'],
},
```

add npm package
---------------

1. [Webpack - Rails (Webpacker)]({% post_url 2018-05-23-webpack-rails-webpacker %})

say, we want to use `toastr` JS library which consists of CSS and JS files.

- install npm package

  ```sh
  $ cd assets
  $ yarn install toastr
  ```

- import CSS

  ```scss
  // assets/css/app.scss

  // resolves to assets/node_modules/toastr/build/toastr.min.js
  @import '~toastr/build/toastr.min';
  ```

  when importing Sass files, `~` prefix in front of imported file is used to
  tell Sass loader that this is not a relative import - it's relative in fact
  but relative to _node\_modules/_ rather than current directory:

  > <https://github.com/webpack-contrib/sass-loader#imports>
  >
  > You can import your Sass modules from node_modules. Just prepend them
  > with a ~ to tell webpack that this is not a relative import:
  >
  > @import "~bootstrap/dist/css/bootstrap";
  >
  > It's important to only prepend it with ~, because ~/ resolves to the home
  > directory. webpack needs to distinguish between bootstrap and ~bootstrap
  > because CSS and Sass files have no special syntax for importing relative
  > files. Writing `@import "file"` is the same as `@import "./file"`.

- import JS

  ```javascript
  // assets/js/app.js

  // resolves to assets/node_modules/toastr/build/toastr.min.js
  import Toastr from 'toastr/build/toastr.min';

  document.addEventListener('DOMContentLoaded', () => {
    Toastr.info('Hello world!');
  });
  ```

### add jQuery

```sh
$ cd assets
$ yarn install jquery
```

just like for Webpacker, there are 2 ways to use jQuery
(see the post about Webpacker for details and links):

- use `ProvidePlugin` to load jQuery automatically

  ```diff
    // assets/webpack.config.js

    const path = require('path');
  + const webpack = require('webpack');

    // ...

    plugins: [
      new MiniCssExtractPlugin({filename: 'css/[name].css'}),
      new CopyWebpackPlugin(['static/']),
  +   new ProvidePlugin({$: 'jquery', jQuery: 'jquery'}),
    ],
  ```

- *[RECOMMENDED]* import jQuery manually in each module where it's used

  ```javascript
  // assets/js/app.js

  import $ from 'jquery';
  ```

### add Bootstrap

```sh
$ cd assets
$ yarn add bootstrap
```

```scss
// assets/css/app.scss

@import '~bootstrap/dist/css/bootstrap.min';
```

### React

```sh
$ cd assets
$ yarn add react react-dom prop-types
$ yarn add babel-preset-react --dev
```

```diff
  // assets/.babelrc

  "presets": [
    // ...
+   "react"
  ]
```

```diff
  // assets/webpack.config.js

  resolve: {
-   extensions: ['.js', '.css', '.sass', '.scss'],
+   extensions: ['.js', '.jsx', '.css', '.sass', '.scss'],
  },
```

add Webpack development server
------------------------------

1. <https://github.com/webpack/docs/wiki/webpack-dev-server>

there are 2 major development servers for Webpack (avalable as npm packages):

- `webpack-dev-server`
- `webpack-serve`

though `webpack-serve` seems to be a successor of `webpack-dev-server` and
its faster alternative (and the latter is in a maintenance-only mode now)
I haven't managed to make it work - so use `webpack-dev-server` for now:

```sh
$ cd assets
$ yarn add webpack-dev-server --dev
```

it's possible to configure `webpack-dev-server` in Webpack config:

```javascript
// assets/webpack.config.js

optimization: { /* ... */ },
// https://webpack.js.org/configuration/dev-server/#devserver
// webpack-dev-server options
devServer: {
  host: 'localhost',
  // port 3035 is used by Webpacker by default
  port: 3045,
  // https://webpack.js.org/configuration/watch/#watchoptions-ignored
  watchOptions: {ignored: /node_modules/},
},
```

BTW it's possible to see what assets are served by `webpack-dev-server`
(and to make sure it's working) right in browser by opening configured URL:

- <http://localhost:3045>

  lists all files in _assets/_ directory - say, _js/app.js_ is replaced with
  corresponding output bundle while _css/app.scss_ is left untouched (as CSS
  is not extracted into output bundles in development).

- <http://localhost:3045/webpack-dev-server>

  lists all assets served by `webpack-dev-server` including _*.hot-update.json_
  files used for HMR which aren't shown when `http://localhost:3045` is opened.

### Webpack development server vs. Webpack in watch mode

both are meant to be used in development environment only - output bundles
for production environment are compiled using `deploy` script.

both rebuild output bundles on file changes - this is the only function of
Webpack in watch mode.

in addition Webpack development server:

- serves output bundles
- reloads the whole page (when using Live Reload) or updated module only
  (when using Hot Module Replacement aka HMR)

  > <https://github.com/webpack/docs/wiki/webpack-dev-server#automatic-refresh>
  >
  > In Hot Module Replacement, the bundle is notified that a change happened.
  > Rather than a full page reload, a Hot Module Replacement runtime could
  > then load the updated modules and inject them into a running app.

- allows not to include `[hash]` substitutions in filenames (moreover -
  they are prohibited)

  NOTE: it's not relevant when using Phoenix - hashes are not used anyway.

  > <https://github.com/webpack/webpack-dev-server/issues/438>
  >
  > There is no need to use hash (bundle names with [hash] in file name) in
  > webpack dev server, there are no cache problems.

  > <https://github.com/webpack/webpack-dev-server/issues/377#issuecomment-241258405>
  >
  > You should not use [chunkhash] or [hash] for development. This will
  > cause many other issues, like a memory leak, because the dev server
  > does not know when to clean up the old files.

**summary**

use Webpack development server instead of Webpack in watch mode.

### Webpack in watch mode (for reference only)

Webpack is run in watch mode when it's passed these options:

```
$ cd assets
$ node_modules/.bin/webpack --help
...
--watch, -w         Enter watch mode, which rebuilds on file change. [boolean]
--watch-stdin, --stdin     Stop watching when stdin stream has ended [boolean]
```

`--watch-stdin` option must be used instead of `--watch` - or else Webpack
process (when run as watcher) is not killed when Phoenix server shuts down
and you'll end up with an orphaned Node.js process.

add watch script
----------------

```
$ cd assets
$ node_modules/.bin/webpack-dev-server --help
...
--mode                   Enable production optimizations or development hints.
--hot                    Enables Hot Module Replacement            [boolean]
--watch-stdin, --stdin   close when stdin ends                     [boolean]
```

```diff
  // assets/package.json

  {
    "scripts": {
-     "deploy": "webpack --mode production"
+     "deploy": "webpack --mode production",
+     "watch": "webpack-dev-server --mode development --hot --watch-stdin"
    },
  }
```

pass `--watch-stdin` option for the same reason that it's passed to Webpack
in watch mode - to kill a watcher running `webpack-dev-server` when Phoenix
server shuts down.

add watcher
-----------

1. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
2. <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#bace>
3. <https://til.hashrocket.com/posts/frbeappww3-phoenix-will-watch-your-js-for-you-just-watch>

> <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
>
> :watchers - a set of watchers to run alongside your server.

watcher is added for development environment only (it's possible to specify
`watch` script or `webpack-dev-server` executable with options directly):

```elixir
# config/dev.exs

config :my_app, MyAppWeb.Endpoint,
  # ...
  watchers: [yarn: ["run", "watch", cd: Path.expand("../assets", __DIR__)]]
```

link output bundles and static assets
-------------------------------------

1. <https://medium.com/@kimlindholm/adding-webpack-3-to-phoenix-e6633dbc2bc4#68ec>
2. <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#ec1e>

create a separate module for Webpack helpers:

```elixir
# lib/my_app_web/views/webpack_helpers.ex

defmodule MyAppWeb.WebpackHelpers do
  # for static_pach/2
  import SithexWeb.Router.Helpers

  def js_script_tag(conn) do
    ~s(<script src=#{webpack_path(conn, "/js/app.js")}></script>)
  end

  def css_link_tag(conn) do
    if Mix.env == :prod do
      ~s(link rel="stylesheet" href="#{static_path(conn, "/css/app.css")}")
    else
      # CSS is not extracted into separate files in development
      ""
    end
  end

  def webpack_path(conn, path) do
    if Mix.env == :prod do
      static_path(conn, path)
    else
      # all assets (including output bundles) are served with
      # `webpack-dev-server` in development
      "http://localhost:3045#{path}"
    end
  end
end
```

import Webpack helpers for all views:

```elixir
# lib/my_app_web.ex

def view do
  quote do
    # ...

    import SithexWeb.Router.Helpers
    import SithexWeb.ErrorHelpers
    import SithexWeb.Gettext
    import SithexWeb.WebpackHelpers
  end
end
```

use `css_link_tag/1` and `js_script_tag/1` helpers to include output bundles
in layout:

```slim
/ lib/my_app_web/templates/layout/app.html.slime

doctype html
html lang="en"
  head
    / ...
    title MyApp
    / => {:safe, css_link_tag(@conn)}
    = raw(css_link_tag(@conn))

  body
    / ...
    / => {:safe, js_script_tag(@conn)}
    = raw(js_script_tag(@conn))
```

use `webpack_path/2` helper to reference static assets (images, etc.):

```slim
/ => img src=webpack_path(@conn, "/images/foo.jpg")
= img_tag(webpack_path(@conn, "/images/foo.jpg"))
```

also it's NOT necessary to require all images in _assets/static/images/app.js_
recursively and import this file later in _assets/js/app.js_ like in Webpacker
(since we copy all of them manually using `CopyWebpackPlugin`).

### HMR

configure output in Webpack config for HMR to work (Live Reload would work
without these changes):

```diff
  // assets/webpack.config.js

-   output: {
-     path: path.resolve(__dirname, '../priv/static'),
-     filename: 'js/[name].js',
-   },
+   output: devMode
+     ? {
+       // IDK why `public` - it's the only path that works
+       path: path.resolve(__dirname, 'public'),
+       filename: 'js/[name].js',
+       // trailing slash is required
+       publicPath: 'http://localhost:3045/',
+     }
+     : {
+       path: path.resolve(__dirname, '../priv/static'),
+       filename: 'js/[name].js',
+     },
    // ...
    devServer: {
      host: 'localhost',
      port: 3045,
+     // this header is also required for HMR to work
+     headers: {
+       'Access-Control-Allow-Origin': '*',
+     },
      watchOptions: {ignored: /node_modules/},
    },
```

unless configured this way, you'll get this error in server log (or no errors
at all but HMR still wouldn't work):

```
GET /710cab78ba975de05092.hot-update.json
** (Phoenix.Router.NoRouteError) no route found for GET /710cab78ba975de05092.hot-update.json (MyAppWeb.Router)
```

NOTE: this file can be found at <http://localhost:3045/webpack-dev-server>.

### static_path/1 helper

1. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#c:static_path/1>
2. <https://github.com/webpack/webpack/issues/86>
3. <https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Digest.html>

you cannot reference hardcoded paths like _/js/app.js_ or _/css/app.css_ inside
layout file directly because output bundle names will most likely contain some
kind of hashes in production environment - in Phoenix these hashes are appended
to static files by `phx.digest` Mix task.

without Phoenix you would use something like `webpack-file-changer` plugin that
changes file path dynamically in layout file during asset compilation.

Webpacker, for instance, provides special helpers:

- `javascript_pack_tag` and `stylesheet_pack_tag`

  they are used to link output bundles (packs) in layout file.

- `asset_pack_path`

  it's used to reference assets in views.

under the hood all these helpers parse manifest file to find actual file paths.

just like Webpacker, Phoenix provides `static_path/1` helper which generates
routes to static files in _priv/static/_. this helper is not aware of Webpack
but uses cache static manifest to find actual file paths (much like Webpacker
helpers do).
