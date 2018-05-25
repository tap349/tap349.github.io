---
layout: post
title: Webpack - Rails (Webpacker)
date: 2018-05-23 02:31:57 +0300
access: public
comments: true
categories: [webpack, rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/rails/webpacker>
2. <https://github.com/rails/webpacker/tree/master/docs>
3. <https://www.neontsunami.com/posts/replacing-rails-asset-pipeline-with-webpacker>

> <https://news.ycombinator.com/item?id=15883425>
>
> Webpacker is an anti-pattern: pollute the ruby sphere with JS and pollute
> the JS sphere with Ruby because you don't wanna read the Webpack tutorial.

notes
-----

### environments

1. <https://github.com/rails/webpacker/blob/master/docs/webpack.md>
2. <https://github.com/rails/webpacker>

`NODE_ENV` values:

- production
- development

assets are compiled in `NODE_ENV` mode but use/load `RAILS_ENV` configuration
(production environment is used as a fallback one) from _config/webpacker.yml_.

> binstubs compile in development mode, rake tasks compile in production mode:

```ruby
# compiles in development mode unless NODE_ENV is specified
./bin/webpack
./bin/webpack-dev-server

# compiles in production mode by default unless NODE_ENV is specified
bundle exec rails assets:precompile
bundle exec rails webpacker:compile
```

### packs

1. <https://github.com/rails/webpacker/blob/master/docs/assets.md>

pack (Webpacker term) == entry (Webpack term)

> <https://github.com/rails/webpacker/blob/master/docs/folder-structure.md#packs-aka-webpack-entries>
>
> Packs a.k.a webpack entries
>
> "Packs" is a special directory made only for webpack entry files so don't
> put anything here that you don't want to link in your views.

> <https://github.com/rails/webpacker/issues/581#issuecomment-316937091>
>
> 'packs' is a special directory meant only for entry files i.e. written either
> in JS, Coffee or TS (as long as it compiles to JS). Every other thing should
> be outside packs and may live under app/javascript or anywhere you wish to
> be as long as you can reference it. If you reference styles in your pack all
> styles will be extracted to [pack_name].css, which you can reference in your
> views.

> <https://github.com/rails/webpacker/issues/471#issuecomment-306115940>
>
> by convention one should only put .js packs inside packs directory since they
> are treated in a special way by webpack. Other types of files like - styles,
> fonts and images should be placed outside of the packs directory and can be
> referenced from there.

> <https://github.com/rails/webpacker/blob/master/docs/assets.md#import-from-node-modules>
>
> Please note that your styles will always be extracted into [pack_name].css

=> only JS files are allowed to be packs (entries), CSS files imported inside
packs will be extracted into _\<pack\_name>.css_ file which can be referenced
in layout with `stylesheet_pack_tag` helper.

### assets

1. <https://webpack.js.org/guides/dependency-management/#require-context>

assets can be imported one by one using `import` statements or recursively
using `require.context()` function:

```javascript
// import each image manually
import './foo.jpg';

// or all images in directory recursively
require.context('images/', true, /\.(gif|jpeg|jpg|png|svg)$/i);
```

### loaders

> <https://github.com/rails/webpacker/blob/master/docs/webpack.md#loaders>
>
> You can also modify the loaders that Webpacker pre-configures for you.

basic loaders are installed as dependencies of `@rails/webpacker` NPM package:

```conf
# yarn.lock

"@rails/webpacker@3.5":
  version "3.5.3"
  dependencies:
    babel-loader "^7.1.4"
    css-loader "^0.28.11"
    file-loader "^1.1.11"
    postcss-loader "^2.1.4"
    sass-loader "^6.0.7"
    style-loader "^0.20.3"
```

these loaders are also pre-configured by Webpacker (that is rules for loader
modules are added to _webpack.config.js_ Webpack config that must be generated
by Webpacker under the hood).

print `environment.loaders` property inside _config/webpack/environment.js_
for the list of configured loaders:

```diff
  // config/webpack/environment.js

  const {environment} = require('@rails/webpacker');

+ console.log(environment.loaders);
  module.exports = environment;
```

configuration
-------------

### plugins

- `babel-plugin-module-resolver`

  1. <https://github.com/rails/webpacker/blob/master/docs/assets.md#using-babel-module-resolver>

  > <https://rossta.net/blog/from-sprockets-to-webpack.html#resolving-application-modules>
  >
  > As a convenience, we learned to resolve modules in our application as import
  > 'some_module' instead of via relative paths like import '../some_module. To
  > do this, we set up an alias in .babelrc. Webpacker installs .babelrc as a
  > separate configuration file for Babel. We added the babel-plugin-module-resolver
  > and updated the relevant section in our .babelrc in the plugins section.

  `babel-plugin-module-resolver` is used to reference assets relative to fixed
  root - say, _app/assets/_:

  ```sh
  $ yarn add babel-plugin-module-resolver
  ```

  ```diff
    // .babelrc

    {
      // ...
      "plugins": [
        "syntax-dynamic-import",
        "transform-object-rest-spread",
  -     ["transform-class-properties", { "spec": true }]
  +     ["transform-class-properties", { "spec": true }],
  +     ["module-resolver", {
  +       "root": ["./app/assets"]
  +     }]
      ]
    }
  ```

  maybe using this plugin is not required since I can reference _css/app.js_
  and _js/app.js_ inside my pack file (_app/assets/packs/app.js_) - that is
  Webpacker understands that these paths are relative to `source_entry_path`
  (_app/assets/_ - see _config/webpacker.yml_) rather than relative to pack
  file.

### source directory

1. <https://github.com/rails/webpacker#paths>
2. <https://github.com/rails/webpacker/blob/master/docs/folder-structure.md#source>

change source directory to _app/assets/_:

```diff
  # config/webpacker.yml

  default: &default
-   source_path: app/javascript
+   source_path: app/assets
    source_entry_path: packs
    public_output_path: packs
    cache_path: tmp/cache/webpacker
```

### pack

1. <https://github.com/rails/webpacker/blob/master/docs/assets.md#import-from-node-modules>

import all assets in pack file:

```javascript
// app/assets/packs/app.js

import 'css/app';
import 'images/app';
import 'js/app';
```

see next sections for details on how to import specific assets.

packs then included in layout file using `javascript_pack_tag` and
`stylesheet_pack_tag` helpers:

> <https://hype.codes/how-assemble-rails-frontend-using-webpacker>
>
> Very similar to the Asset Pipeline. Use new javascript_pack_tag and
> stylesheet_pack_tag helpers. They will add necessary HTML tags with
> links to assembled packs.

```slim
/ app/views/layouts/application.html.slim

/ app.js
= javascript_pack_tag('app')
/ app.css
= stylesheet_pack_tag('app')
```

### styles

1. <https://github.com/rails/webpacker/blob/master/docs/css.md>

- add Sass support

  Sass loader is already installed as dependency of and pre-configured
  by Webpacker - almost no additional configuration is required.

- rename _app/assets/css/app.css_ to _app.sass_ or _app.scss_

  no changes in pack file are required:

  ```javascript
  // app/assets/packs/app.js

  import 'css/app';
  import 'images/app';
  import 'js/app';
  ```

  the point is that Webpacker tries to use all extensions listed in its
  config - just make sure `sass` or `scss` extension is listed there:

  ```yaml
  # config/webpacker.yml

  default: &default
    # ...
    extensions:
      # ...
      - .sass
      - .scss
  ```

- add `resolve-url-loader` package

  1. <https://github.com/rails/webpacker/issues/1267>

  > <https://github.com/rails/webpacker/blob/master/docs/css.md#resolve-url-loader>
  >
  > Since Sass/libsass does not provide url rewriting, all linked assets
  > must be relative to the output. Add the missing url rewriting using
  > the resolve-url-loader. Place it directly after the sass-loader in
  > the loader chain.

  ```diff
    # config/webpack/environment.js

    const {environment} = require('@rails/webpacker');

  + environment.loaders.get('sass').use.splice(-1, 0, {
  +   loader: 'resolve-url-loader',
  +   options: {
  +     attempts: 1
  +   }
  + });
  ```

  also they say (see the link) URL rewriting is required to import
  Bootstrap with Webpacker.

  ***UPDATE***

  when using `resolve-url-loader`, there's an error when importing Bootstrap:

  ```
  $ bin/webpack
  ...
  (Emitted value instead of an instance of Error)   resolve-url-loader cannot operate: CSS error
    ENOENT: no such file or directory, open '<app_dir>/app/assets/css/bootstrap.css.map'
  ```

  that is _bootstrap.css.map_ is getting resolved relative to current
  directory (Bootstrap styles are imported in _app/assets/css/app.scss_)
  rather than relative to _node\_modules/bootstrap/dist/css/_.

  error is gone when `resolve-url-loader` package is removed => so don't
  use it so far.

- add Bootstrap

  1. <https://github.com/rails/webpacker/blob/master/docs/css.md#add-bootstrap>
  2. <https://github.com/rails/webpacker/issues/1267>

  don't add `resolve-url-loader` package so far - it works without it and
  doesn't work when it's added (see above).

  ```sh
  $ yarn add bootstrap
  ```

  ```diff
    // app/assets/css/app.scss

  + @import '~bootstrap/dist/css/bootstrap';
  ```

  when importing Sass files, `~` prefix in front of imported file is used to
  tell that this is not a relative import: it's relative in fact but relative
  to _node\_modules/_ rather than current directory:

  ```scss
  // resolves to node_modules/bootstrap/dist/css/bootstrap
  @import '~bootstrap/dist/css/bootstrap';
  ```

  > <https://github.com/rails/webpacker/issues/454#issuecomment-305764527>
  >
  > You see, sass-loader needs a ~ in the beginning to understand that a
  > particular file is using a node_modules import.

  > <https://www.neontsunami.com/posts/replacing-rails-asset-pipeline-with-webpacker>
  >
  > it's worth learning about the ~ character when using imports as it will
  > start a lookup from the node_modules directory.

### images

1. <https://github.com/rails/webpacker/blob/master/docs/assets.md#link-in-your-rails-views>
2. <https://github.com/rails/webpacker/issues/705#issuecomment-325394754>
3. <https://github.com/rails/webpacker/issues/1101>
4. <https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79#46fd>
5. <https://stackoverflow.com/questions/49222136>

```diff
  // app/assets/packs/app.js

  import 'css/app';
+ import 'images/app';
  import 'js/app';
```

```javascript
// app/assets/images/app.js

require.context('images/', true, /\.(gif|jpeg|jpg|png|svg)$/i);
```

make sure specified image extensions are listed in Webpacker config:

```yaml
# config/webpacker.yml

default: &default
  # ...
  extensions:
    # ...
    - .gif
    - .jpeg
    - .jpg
    - .png
    - .svg
```

use `asset_pack_path` helper to get image URL in views:

```slim
/ app/views/test/show.html.slim

img src=asset_pack_path('images/foo.jpg')
/ => /packs/images/foo-095e9f86a13789b1d0b86b9b5a9ff94a.jpg
```

### Procfile

1. <https://github.com/rails/webpacker#development>

> <https://github.com/rails/webpacker#development>
>
> This process will watch for changes in the app/javascript/packs/*.js
> files and automatically reload the browser to match.
>
> Once you start this development server, Webpacker will automatically
> start proxying all webpack asset requests to this server. When you
> stop the server, it'll revert back to on-demand compilation.

use _Procfile_ to start Rails server and Webpacker development server:

```
# Procfile

# for some reason port 5000 is used by default
rails: rails server -p 3000
webpack: bin/webpack-dev-server
```

BTW when running Webpacker development server (`webpack-dev-server`)
you don't have to reload page constantly after changing asset files:

> <https://github.com/rails/webpacker#development>
>
> This process will watch for changes in the app/javascript/packs/*.js
> files and automatically reload the browser to match.

### Yarn integrity check

Yarn integrity check might produce false positives:

```sh
$ yarn install
$ rails assets:precompile
...
========================================
  Your Yarn packages are out of date!
  Please run `yarn install` to update.
========================================
```

disable Yarn integrity check in development environment
(it's disabled in production environment by default):

```ruby
# config/environments/development.rb

Rails.application.configure do
  config.webpacker.check_yarn_integrity = false
  # ...
end
```

deployment
----------

1. <https://github.com/rails/webpacker/blob/master/docs/deployment.md>

> <https://github.com/rails/webpacker/blob/master/docs/deployment.md>
>
> Webpacker hooks up a new webpacker:compile task to assets:precompile,
> which gets run whenever you run assets:precompile. If you are not using
> Sprockets webpacker:compile is automatically aliased to assets:precompile.

NOTE: _public/assets/_ directory is created automatically in _shared/_
      by Capistrano even if it's not listed in `linked_dirs`.

Rake tasks compile in production mode by default (see `about environments`
section above) - no need to set `NODE_ENV=production` manually when running
`assets:precompile` task.

### Capistrano

1. [Webpack - Troubleshooting]({% post_url 2018-05-22-webpack-troubleshooting %})
2. <https://rossta.net/blog/from-sprockets-to-webpack.html#deploying-with-capistrano-and-nginx>

```ruby
# Capfile

require 'capistrano/rails/assets'
```

set correct assets prefix so that `deploy:assets:backup_manifest` task doesn't
fail when performing deployment (see `Webpack - Troubleshooting` page):

```ruby
# config/deploy.rb

set :assets_prefix, 'packs'
```

link _public/packs/_ and _node\_modules/_ directories:

> <https://rossta.net/blog/from-sprockets-to-webpack.html#deploying-with-capistrano-and-nginx>
>
> set public/packs and node_modules as shared directories to ensure Webpack
> build output and NPM package installation via Yarn are shared across deploys

```diff
  # config/deploy.rb

- append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets'
+ append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'public/packs', 'node_modules'
```