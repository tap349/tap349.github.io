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

it's assumed that it's not React-only project - React is integrated
into existing application with traditional HTML templates.

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <http://whatdidilearn.info/2018/05/20/how-to-use-webpack-and-react-with-phoenix-1-3.html#configure-react>

install npm packages
--------------------

```sh
$ cd assets
$ yarn add react react-dom prop-types babel-preset-react
```

NOTE: add `babel-preset-react` package as normal (not development) dependency or
      else it may be not installed in production if installing npm packages with
      `yarn install --production` command.

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

don't install `@babel/preset-react` package - it requires Babel 7
(while Babel 6 is currently used in my project).

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

add HTML element with `react` ID to existing page template
----------------------------------------------------------

of course you can choose any other ID name and insert this
element wherever you want inside the template:

```slim
// lib/my_app_web/templates/user/new.html.slime

// it's better to serialize user outside the template
// (say, in a view helper)
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

troubleshooting
---------------

### [Rails] ReferenceError: Unknown option: ...react/index.js.Children

```
$ cap production deploy
...
01 ERROR in ./app/assets/packs/app.js
01 Module build failed: ReferenceError: [BABEL]
  /home/sith/production/releases/20180626151539/app/assets/packs/app.js:
  Unknown option: /home/sith/production/releases/20180626151539/node_modules/react/index.js.Children.
  Check out http://babeljs.io/docs/usage/options/ for more information about options.
```

**solution**

1. <https://github.com/rails/webpacker/issues/1330>
2. <https://github.com/rails/webpacker/issues/1460>
3. <https://github.com/rails/webpacker/issues/1441#issuecomment-383328538>
4. <https://github.com/rails/webpacker/issues/1037#issuecomment-347610374>

> <https://stackoverflow.com/a/50659957>
>
> The error is unhelpful, but the issue is that your config has react in
> the preset list, but it can't find the babel-preset-react module in your
> node_modules, so instead it is loading the react module itself as if it
> were a preset. But since the "react" module isn't a preset, Babel throws.

`babel-preset-react` npm package is added as development dependency to
_package.json_ - it's removed by `webpacker:yarn_install` task which is
run as depedency of `webpacker:compile` task:

> <https://github.com/rails/webpacker/blob/master/docs/deployment.md>
>
> Webpacker hooks up a new webpacker:compile task to assets:precompile,
> which gets run whenever you run assets:precompile.

`webpacker:yarn_install` runs this command:

```sh
yarn install --no-progress --frozen-lockfile --production
```

`--production` flag means that development dependencies won't be installed.

surprisingly `babel-preset-react` package is not installed even when it's
added as normal depedency - maybe it's some caching problem or bug in Yarn
itself.

my current solution (workraround to be precise) is to run `yarn install`
manually (without `--production` flag) and skip `webpacker:yarn_install`
task at all:

```ruby
# lib/tasks/webpacker.rake

Rake::Task['webpacker:yarn_install'].clear

namespace :webpacker do
  desc 'Skip default webpacker yarn install'
  task :yarn_install do
    puts 'Skipping webpacker yarn install'
  end
end
```

```diff
  # config/deploy.rb

  namespace :deploy do
+   task :yarn_install do
+     on roles(:app) do
+       within release_path do
+         execute "cd #{release_path} && yarn install"
+       end
+     end
+   end

    # ...
  end

+ before 'deploy:assets:precompile', 'deploy:yarn_install'
```

or else it's possible to compile assets locally and copy them to production
server with rsync ([link](https://stackoverflow.com/a/45236293)) - this way
you don't have to install Node.js and Yarn on production server at all.

***UPDATE***

In the end I reverted changes above and started to precompile assets locally.
