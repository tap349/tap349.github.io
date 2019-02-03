---
layout: post
title: Elixir - Publish Hex Package
date: 2019-02-03 21:18:52 +0300
access: private
comments: true
categories: [elixir]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://hex.pm/docs/publish>

### register Hex user

```sh
$ mix hex.user register
...
Username: tap349
Email: alexey.terekhov.tap349@gmail.com
Account password:
Account password (confirm):
Registering...
Generating keys...
You have authenticated on Hex using your account password. However, Hex requires you to have a local password that applies only to this machine for security purposes. Please enter it.
Local password:
Local password (confirm):
You are required to confirm your email to access your account, a confirmation email has been sent to alexey.terekhov.tap349@gmail.com
```

### submit package

```sh
$ mix hex.publish
Building ecto_cqrs 0.1.0
  Dependencies:
    ecto_sql ~> 3.0 (app: ecto_sql)
    postgrex >= 0.0.0 (app: postgrex)
  App: ecto_cqrs
  Name: ecto_cqrs
  Files:
    lib
    lib/ecto_cqrs
    lib/ecto_cqrs/mutator.ex
    lib/ecto_cqrs/query.ex
    lib/ecto_cqrs/helpers.ex
    lib/ecto_cqrs/loader.ex
    lib/ecto_cqrs.ex
    .formatter.exs
    CHANGELOG.md
    LICENSE.md
    README.md
    mix.exs
  Version: 0.1.0
  Build tools: mix
  Description: CQRS library for Ecto
  Licenses: MIT
  Links:
    github: https://github.com/tap349/ecto_cqrs
  Elixir: ~> 1.8
Before publishing, please read the Code of Conduct: https://hex.pm/policies/codeofconduct

Publishing package to public repository hexpm.
Proceed? [Yn]
Building docs...
...
==> ecto_cqrs
Compiling 5 files (.ex)
Generated ecto_cqrs app
Docs successfully generated.
View them at "doc/index.html".
Local password:
Publishing package...
[#########################] 100%
Package published to https://hex.pm/packages/ecto_cqrs/0.1.0 (5fdac36517f847dca30623ee922c6b315676ccd0e8eb07561fedc304d0ffe268)
Publishing docs...
[#########################] 100%
Docs published to https://hexdocs.pm/ecto_cqrs/0.1.0
```
