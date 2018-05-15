---
layout: post
title: Rails - Troubleshooting
date: 2018-03-29 17:02:54 +0300
access: public
comments: true
categories: [rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

null array elements are removed in params
-----------------------------------------

1. <https://gist.github.com/subfuzion/08c5d85437d5d4f00e58#post>

data is sent in JSON format via POST request.

`null` array elements are present when reading request body directly:

```ruby
# in controller
request.body.read
```

but they are already removed in parsed params:

```ruby
# in controller
params.to_unsafe_h
```

**solution**

1. <https://stackoverflow.com/a/25428800/3632318>
2. <https://github.com/rails/rails/commit/e8572cf2f94872d81e7145da31d55c6e1b074247>

probably this has something to do with `perform_deep_munge` Rails feature -
you might try to disable it:

```ruby
config.action_dispatch.perform_deep_munge = false
```

but I've chosen to send `null` array elements as empty strings - they are
not removed by Rails and are properly converted to `nil`s by `Form` schema
of dry-validation.

Could not load 'guard/rails' or '~/.guard/templates/rails' or find class Guard::Rails
-------------------------------------------------------------------------------------

```
$ guard init rails
ERROR - Could not load 'guard/rails' or '~/.guard/templates/rails' or find class Guard::Rails
```

**solution**

don't run `guard init rails` - I have no `guard-rails` gem in _Gemfile_.
follow instructions from <https://github.com/guard/guard-rspec> instead
to create _Guardfile_:

```sh
$ guard init rspec
```

ActionController::InvalidAuthenticityToken
------------------------------------------

1. <https://github.com/omniauth/omniauth/issues/237#issuecomment-908481>
2. <https://stackoverflow.com/questions/941594/understanding-the-rails-authenticity-token>

it's possible to disable CSRF for API requests altogether either in
`ApplicationController` or specific controller with:

```ruby
skip_before_action :verify_authenticity_token
```

<https://security.stackexchange.com/a/166798>:

> Browsers send cookies along with all requests. CSRF attacks depend upon
> this behavior. If you do not use cookies, and don't rely on cookies for
> authentication, then there is absolutely no room for CSRF attacks, and
> no reason to put in CSRF protection. If you have cookies, especially if
> you use them for authentication, then you need CSRF protection.

request is nil in ActionController::ParamsWrapper
-------------------------------------------------

error was caused by having `request` action in controller which returned
nil (this action must have overriden corresponding Rails method) => don't
use `request` name for any methods inside controllers.

`rails credentials:edit` opens empty file every time
----------------------------------------------------

1. <https://github.com/rails/rails/issues/31286>

whatever I type in this file is not saved to _config/credentials.yml.enc_.

**solution**

use terminal-based editor instead of my default editor MacVim:

```sh
$ EDITOR=vim rails credentials:edit
$ rails credentials:show
```
