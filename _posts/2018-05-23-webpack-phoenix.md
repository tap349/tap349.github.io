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

### `--watch-stdin` option

```
$ assets/node_modules/webpack/bin/webpack.js --help
...
--watch-stdin, --stdin     Stop watching when stdin stream has ended [boolean]
```

use `--watch-stdin` option instead of `--watch` one - or else Webpack process
(watcher running `watch` script from _assets/package.json_) is not killed when
Phoenix server shuts down and you'll end up with an orphaned Node.js process.

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
    // entry point chunk name is 'main' by default
    entry: {app: 'js/app.js'},
    output: {
      path: path.resolve(__dirname, '../priv/static'),
      filename: 'js/[name]-[chunkhash].js',
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
        // MiniCssExtractPlugin is not used in dev mode
        // (no need to specify filename without hash for dev mode)
        //
        // filename is relative to output.path
        // IDK when and how chunkFilename is used so I don't set it here
        //
        // https://github.com/webpack-contrib/mini-css-extract-plugin#long-term-caching:
        // use [contenthash] instead of [hash] for long term caching.
        //
        new MiniCssExtractPlugin({filename: 'css/[name]-[hash].css'}),
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

- `style-loader`

  `style-loader` doesn't support generating CSS source map at all -
  use `MiniCssExtractPlugin` instead.

  still `MiniCssExtractPlugin` doesn't support HMR yet - so use `style-loader`
  in development mode.

- `devtool` Webpack option

  `devtool` option has effect on generating CSS source map in development
  mode only - set it to any `source-map`-like value to generate CSS source
  map in development mode.

  CSS source map is always generated in production mode (provided
  `map: {inline: false}` CSS processor option is passed - see below).

- `sourceMap` options of Sass and CSS loaders

  `sourceMap` options of both loaders seem to have no effect on generating
  CSS source map.

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
  > files.

  sample _manifest.json_:

  ```json
  {
    "app.css": "css/app-b6889e3643d039f89700.css",
    "app.js": "js/app-14822afe1edfebfb58bb.js"
  }
  ```

### source file name of CSS source map

TODO: for some reason CSS source map has chunk name when running
      webpack in development mode (and using MiniCssExtractPlugin).

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

IDK who to blame for this behaviour and how to change it - the only difference
with Webpacker is that the latter uses `extract-text-webpack-plugin` instead of
`mini-css-extract-plugin` to extract CSS files.

copy static assets
------------------

TODO: CopyWebpackPlugin?
      (http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html)
      what about adding hashes? file-loader does it?

add Bootstrap
-------------

TODO

Webpack in watch mode vs. Webpack development server
----------------------------------------------------

both are meant to be used in development environment only - output bundles
for production environment are compiled using `deploy` script.

### Webpack in watch mode

TODO: running `watch` doesn't compile CSS assets and doesn't create source
      maps (it outputs JS bundle and manifest only) - it doesn't create when
      webpack is run in `development` mode only (this is because style-loader
      is used, source maps are probably never created in dev mode - something
      like `eval` must be used for devtool in dev mode).

```
$ assets/node_modules/webpack/bin/webpack.js --help
...
--watch, -w  Enter watch mode, which rebuilds on file change.        [boolean]
```

Webpack running in watch mode:

- rebuilds output bundles on file changes
- doesn't reload anything

### Webpack development server

TODO: helpers (see the link above or find other tutorials) - I guess it's
      easier to use webpack-dev-server all the time (not for the sake of HMR
      but because otherwise we've got to create some helper to extract actual
      output bundles from manifest to include them in layout file).

Webpack development server:

- rebuilds output bundles on file changes
- serves output bundles
- reloads the whole page (using Live Reload) or updated module only
  (using Hot Module Replacement aka HMR)

there are 2 major development servers for Webpack (both are available as
separate npm packages):

- `webpack-dev-server`
- `webpack-serve` (more modern and faster alternative to `webpack-dev-server`)

add watcher
-----------

1. <https://hexdocs.pm/phoenix/Phoenix.Endpoint.html#module-runtime-configuration>
2. <https://medium.com/@waffleau/using-webpack-4-with-phoenix-1-3-8245b45179c0#bace>
3. <https://til.hashrocket.com/posts/frbeappww3-phoenix-will-watch-your-js-for-you-just-watch>

add `watch` script in _package.json_ (say, we're using `webpack-serve`
development server to watch for file changes):

```diff
  // assets/package.json

  {
    "scripts": {
-     "deploy": "webpack --mode production"
+     "deploy": "webpack --mode production",
+     "watch": "webpack-serve --config ./webpack.config.js"
    },
  }
```

watcher itself is added for development environment only:

```diff
  # config/dev.exs

  config :my_app, MyAppWeb.Endpoint,
    http: [port: 4000],
    debug_errors: true,
    code_reloader: true,
    check_origin: false,
-   watchers: [],
+   watchers: [yarn: ["run", "watch", cd: Path.expand("../assets", __DIR__)]]
```

add links to output bundles in layout
-------------------------------------

1. <https://elixirforum.com/t/getting-the-features-of-webpack-to-work-with-phoenix-webpack-dev-server-sass-and/13615/3>

TODO: static_path helper in layout
TODO: read the link above
TODO: always use webpack-dev-server because there are no helpers like
      javascript_pack_tag or stylesheet_pack_tag (like in Webpacker)?
      that is Phoenix cannot find /css/app.css and /js/app.js because
      these filenames are with hashes. in webpacker they are implemented
      by looking up actual file names in manifest - use webpack-dev-server
      in phoenix instead (see the link)
