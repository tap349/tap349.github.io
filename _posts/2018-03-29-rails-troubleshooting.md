---
layout: post
title: Rails - Troubleshooting
date: 2018-03-29 17:02:54 +0300
access: public
comments: true
categories: [rails]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

## null array elements are removed in params

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

probably this has something to do with `perform_deep_munge` Rails feature - you
might try to disable it:

```ruby
config.action_dispatch.perform_deep_munge = false
```

but I've chosen to send `null` array elements as empty strings - they are not
removed by Rails and are properly converted to `nil`s by `Form` schema of
dry-validation.

## Could not load 'guard/rails' or '~/.guard/templates/rails' or find class Guard::Rails

```
$ guard init rails
ERROR - Could not load 'guard/rails' or '~/.guard/templates/rails' or find class Guard::Rails
```

**solution**

don't run `guard init rails` - I have no `guard-rails` gem in _Gemfile_. follow
instructions from <https://github.com/guard/guard-rspec> instead to create
_Guardfile_:

```sh
$ guard init rspec
```

## ActionController::InvalidAuthenticityToken

1. <https://github.com/omniauth/omniauth/issues/237#issuecomment-908481>
2. <https://stackoverflow.com/questions/941594/understanding-the-rails-authenticity-token>

it's possible to disable CSRF for API requests altogether either in
`ApplicationController` or specific controller with:

```ruby
skip_before_action :verify_authenticity_token
```

> <https://security.stackexchange.com/a/166798>
>
> Browsers send cookies along with all requests. CSRF attacks depend upon this
> behavior. If you do not use cookies, and don't rely on cookies for
> authentication, then there is absolutely no room for CSRF attacks, and no
> reason to put in CSRF protection. If you have cookies, especially if you use
> them for authentication, then you need CSRF protection.

## request is nil in ActionController::ParamsWrapper

error was caused by having `request` action in controller which returned nil
(this action must have overriden corresponding Rails method) => don't use
`request` name for any methods inside controllers.

## `rails credentials:edit` opens empty file every time

1. <https://github.com/rails/rails/issues/31286>

whatever I type in this file is not saved to _config/credentials.yml.enc_.

**solution**

use terminal-based editor instead of my default editor MacVim:

```sh
$ EDITOR=vim rails credentials:edit
$ rails credentials:show
```

## basic authentication prompt is not shown

the page keeps loading but no prompt is shown:

```
# log/production.log

Started GET "/admin/users" for 109.195.187.178 at 2018-05-22 11:06:29 +0000
Processing by Admin::UsersController#index as HTML
Filter chain halted as :basic_auth rendered or redirected
Completed 401 Unauthorized in 0ms
```

**solution**

I have another application with basic authentication on adjacent subdomain:
maybe, saved credentials for another subdomain and are automatically applied for
current subdomain.

so first I tried to remove entry for another subdomain from `Keychain Access`
application (macOS) but that didn't help.

eventually it turned out the problem was caused by Dashlane Chrome extension - I
managed to get basic authentication prompt once I temporarily disabled the
latter.

## No route matches [GET] "/admin/images/ui-icons_444444_256x240.png"

_log/production.log_:

```
Started GET "/admin/images/ui-icons_444444_256x240.png" for ::1 at 2018-05-22 23:33:20 +0300
ActionController::RoutingError (No route matches [GET] "/admin/images/ui-icons_444444_256x240.png"):
```

this error occurs when any Rails or Phoenix application page is opened - either
in development or production environment.

most likely GET request to fetch `ui-icons_444445_256x240.png` is made when ANY
page is opened - these requests just don't draw attention unless I see them in
server logs of my applications.

**solution**

it has turned out that the culprit is `Clouder` Chrome extension => disable it
so far.

## git stderr: git@github.com: Permission denied (publickey).

```
$ cap production deploy
...
00:00 git:check
      01 git ls-remote git@github.com:complead/iceperk.git HEAD
      01 git@github.com: Permission denied (publickey).
      01 fatal: Could not read from remote repository.
      01
      01 Please make sure you have the correct access rights
      01 and the repository exists.
```

**solution**

```sh
$ ssh-add ~/.ssh/id_rsa
$ cap production deploy
```
