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

# compiles in production mode unless NODE_ENV is specified
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
// app/assets/images/application.js

// import specific image
import './images/foo.jpg';

// all images in directory recursively
require.context('images/', true, /\.(gif|jpeg|jpg|png|svg)$/i);
```

```javascript
// app/assets/javascripts/application.js

// import specific npm module
import Toastr from 'toastr';

// import all JS files in directory recursively
const r = require.context('./pages', true, /\.js$/);
r.keys().forEach(r);
```

### loaders

> <https://github.com/rails/webpacker/blob/master/docs/webpack.md#loaders>
>
> You can also modify the loaders that Webpacker pre-configures for you.

basic loaders are installed as dependencies of `@rails/webpacker` npm package:

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

### Babel

create Babel config:

```javascript
// .babelrc

{
  "presets": [
    [
      "env",
      {
        "modules": false,
        "targets": {
          "browsers": "> 1%",
          "uglify": true
        },
        "useBuiltIns": true
      }
    ]
  ],
  "plugins": [
    "transform-object-rest-spread",
    [
      "transform-class-properties",
      {
        "spec": true
      }
    ]
  ]
}
```

### plugins

- `babel-plugin-transform-runtime`

  1. <https://github.com/babel/babel/issues/5085#issuecomment-374903233>

  `babel-plugin-transform-runtime` is used for `async`/`await` support.

  ```sh
  $ yarn add babel-plugin-transform-runtime --dev
  ```

  ```diff
    // .babelrc

    "plugins": [
      // ...
  +   [
  +     "transform-runtime",
  +     {
  +       "polyfill": false,
  +       "regenerator": true
  +     }
  +   ]
    ]
  ```

- *[SKIP]* `babel-plugin-module-resolver`

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

    "plugins": [
      // ...
  +   [
  +     "module-resolver",
  +     {
  +       "root": [
  +         "./app/assets"
  +       ]
  +     }
  +   ]
    ]
  ```

  maybe using this plugin is not required since I can reference _css/app.js_
  and _js/app.js_ inside my pack file (_app/assets/packs/app.js_) - that is
  Webpacker understands that these paths are relative to `source_entry_path`
  (_app/assets/_ - see _config/webpacker.yml_) rather than relative to pack
  file.

  ***UPDATE***

  this plugin is not required and can be safely removed: modules are resolved
  relative to `source_path` (see Webpacker config) by default.

  I guess Webpacker adds `source_path` to `resolve.modules` under the hood.

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

packs are then included in layout file using `javascript_pack_tag` and
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
  rather than relative to original CSS file (_bootstrap.css_).

  error is gone when `resolve-url-loader` package is removed => so don't
  use it so far - it works without it and doesn't work when it's added.

  **UPDATE**

  in spite of the error above, `resolve-url-loader` might be required
  when using some npm packages like `tabler-ui` - its main CSS file
  contains lots of `url()` statements which are not resolved correctly
  without this loader.

### npm package

1. <https://github.com/rails/webpacker/blob/master/docs/assets.md#import-from-node-modules>
2. <https://gist.github.com/MFry/41fd51e8057152b9b914ecd1379a53d2>

say, we want to use `toastr` JS library which consists of CSS and JS files.

- install npm package

  ```sh
  $ yarn install toastr
  ```

- import CSS

  ```scss
  // app/assets/css/app.scss

  // resolves to node_modules/toastr/build/toastr.min.js
  @import '~toastr/build/toastr.min';
  ```

- import JS

  it's possible to use a standalone module name (say, `toastr`) in `import`
  statement without specifying the full path to the file => this name will
  be resolved to the value of `main` field in _package.json_ of corresponding
  npm package:

  > <https://webpack.js.org/concepts/module-resolution/#module-paths>
  >
  > If the path points to a folder, then the following steps are taken to
  > find the right file with the right extension:
  >
  > If the folder contains a package.json file, then fields specified in
  > resolve.mainFields configuration option are looked up in order, and the
  > first such field in package.json determines the file path.

  ```javascript
  // app/assets/js/app.js

  // resolves to node_modules/toastr/build/toastr.min.js
  import Toastr from 'toastr/build/toastr.min.js';
  // doesn't work
  //import Toastr from 'toastr/build/toastr.min';

  document.addEventListener('DOMContentLoaded', () => {
    Toastr.info('Hello world!');
  });
  ```

  `js` extension is listed in `extensions` section of Webpacker config so it
  must be not necessary to specify it when importing JS files.

  still when filename already contains any extension (_toastr.min_), the file
  is not found - it looks like Webpacker doesn't try to append `js` extension
  in this case.

  there is no such error when using pure Webpack (say, in Phoenix project) -
  so it must be Webpacker issue.

#### jQuery

```sh
$ yarn add jquery
```

there are 2 ways to use jQuery:

- use `ProvidePlugin` to load jQuery automatically

  1. <https://webpack.js.org/plugins/provide-plugin/#usage-jquery>
  2. <https://github.com/rails/webpacker/issues/1389#issuecomment-376991936>
  3. <https://toster.ru/q/281654>

  > <https://stackoverflow.com/a/28989476>
  >
  > Use the ProvidePlugin to inject implicit globals
  >
  > Most legacy modules rely on the presence of specific globals, like jQuery
  > plugins do on $ or jQuery. In this scenario you can configure webpack, to
  > prepend `var $ = require("jquery")` everytime it encounters the global $
  > identifier.

  > <https://webpack.js.org/guides/shimming/>
  >
  > We don't recommend using globals! The whole concept behind webpack is to
  > allow more modular front-end development. This means writing isolated
  > modules that are well contained and do not rely on hidden dependencies
  > (e.g. globals). Please use these features only when necessary.

  ```javascript
  // config/webpack/environment.js

  const webpack = require('webpack');
  const {environment} = require('@rails/webpacker');

  environment.plugins.append(
    'Provide',
    new webpack.ProvidePlugin({$: 'jquery', jQuery: 'jquery'}),
  );

  module.exports = environment;
  ```

- *[RECOMMENDED]* import jQuery manually in each module where it's used

  1. <https://www.npmjs.com/package/jquery#babel>

  ```javascript
  // app/assets/js/app.js

  import $ from 'jquery';
  ```

#### Bootstrap

1. <https://github.com/rails/webpacker/blob/master/docs/css.md#add-bootstrap>
2. <https://github.com/rails/webpacker/issues/1267>

```sh
$ yarn add bootstrap
```

```scss
// app/assets/css/app.scss

@import '~bootstrap/dist/css/bootstrap.min';
```

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

NOTE: don't link _public/packs/_ and _node\_modules/_ directories - this might
      cause problems in the long term.

#### precompile assets on production server

1. [Webpack - Troubleshooting]({% post_url 2018-05-22-webpack-troubleshooting %})
2. <https://rossta.net/blog/from-sprockets-to-webpack.html#deploying-with-capistrano-and-nginx>

```ruby
# Capfile

require 'capistrano/rails/assets'
```

set correct assets prefix so that `deploy:assets:backup_manifest` task doesn't
fail when performing deployment (see `Webpack - Troubleshooting` post):

```ruby
# config/deploy.rb

set :assets_prefix, 'packs'
```

#### precompile assets locally

1. <https://github.com/stve/capistrano-local-precompile>
2. <https://github.com/capistrano/rails/blob/master/lib/capistrano/tasks/assets.rake>
3. <https://github.com/rails/webpacker/blob/master/lib/tasks/webpacker/compile.rake>

currently I precompile assets locally (on development machine or CI server).
also I don't rely on `deploy:assets:precompile` task (which is redefined by
Webpacker) and call `webpacker:compile` task directly.

remove all asset tasks provided by `capistrano-rails` gem:

```diff
  # Capfile

- require 'capistrano/rails/assets'
```

remove `assets_prefix` variable - it's used in `deploy:assets:backup_manifest`
task which is removed now (see above):

```diff
  # config/deploy.rb

- set :assets_prefix, 'packs'
```

add custom asset tasks for Capistrano (they are configured to run
sequentially after `bundler:install` task right in this file):

```ruby
# lib/capistrano/tasks/my_assets.rake

# invoke local task:
#   invoke 'deploy:my_assets:local_precompile'
# invoke external task (with rbenv_prefix):
#   execute :rake, 'assets:clean'
# invoke external task (without rbenv_prefix):
#   execute 'bundle exec rake assets:clean'
namespace :deploy do
  namespace :my_assets do
    desc 'Precompile assets locally'
    task :local_precompile do
      run_locally do
        with rails_env: fetch(:stage) do
          # run yarn manually since webpacker:yarn_install runs yarn
          # with `--production` flag which doesn't install development
          # dependencies while they are required for asset compilation
          #
          # yarn cache is saved to this folder on CI server
          execute 'yarn install --cache-folder ~/.cache/yarn'

          # don't run rake with rbenv_prefix - rbenv is not installed
          # on CI server and it's necessary to symlink its executable
          # to ~/.rbenv/bin/rbenv on macOS locally
          #
          # run webpacker:compile instead of assets:precompile
          # to skip webpacker:yarn_install task at all:
          #
          # Rake::Task.define_task(
          #   "assets:precompile" => [
          #     "webpacker:yarn_install",
          #     "webpacker:compile"
          #   ]
          # )
          execute 'bundle exec rake webpacker:compile'
        end
      end
    end

    desc 'Copy precompiled assets to remote server'
    task :rsync do
      on release_roles(:app) do |server|
        local_path = "#{fetch(:packs_dir)}/"
        remote_path = "#{server.user}@#{server.hostname}:#{release_path}/#{fetch(:packs_dir)}/"

        run_locally do
          execute "#{fetch(:rsync_cmd)} #{local_path} #{remote_path}"
        end
      end
    end

    desc 'Remove all precompiled assets locally'
    task :local_cleanup do
      run_locally do
        with rails_env: fetch(:stage) do
          # public/packs is not a symlink any more - each
          # release has its own public/packs/ directory
          execute "rm -rf #{fetch(:packs_dir)}"
        end
      end
    end
  end
end

namespace :load do
  task :defaults do
    set :packs_dir, 'public/packs'
    set :rsync_cmd, 'rsync -av --delete'

    after 'bundler:install', 'deploy:my_assets:local_precompile'
    after 'bundler:install', 'deploy:my_assets:rsync'
    after 'bundler:install', 'deploy:my_assets:local_cleanup'
  end
end
```

fix CircleCI config to install Yarn and rsync in deploy job:

{% raw %}
```diff
  # .circleci/config.yml

  version: 2
  jobs:
    build:
      # ...

      steps:
        # ...

        - restore_cache: &restore_yarn_cache
            keys:
              - v3-yarn-{{ checksum "yarn.lock" }}
              - v3-yarn-

        # ...

    deploy:
      # ...

      docker:
-       - image: circleci/ruby:2.5.1
-         environment:
-           RAILS_ENV: test
+       # use ruby:2.5.1-node instead of ruby:2.5.1 since now assets
+       # are precompiled locally and `yarn install` command is run
+       # right before asset compilation => Yarn must be installed
+       - image: circleci/ruby:2.5.1-node
+         environment:
+           # RAILS_ENV is set to `production` inside Capistrano tasks
+           #
+           # when it was set to `test` here, assets were compiled to
+           # public/packs-test/ directory for some reason

      steps:
        - checkout

+       - run:
+           name: Install rsync
+           command: sudo apt install rsync

        # ...

        - run:
            name: Bundle install
            command: bundle install --jobs=4 --retry=3 --path vendor/bundle

+       - restore_cache:
+           <<: *restore_yarn_cache
```
{% endraw %}
