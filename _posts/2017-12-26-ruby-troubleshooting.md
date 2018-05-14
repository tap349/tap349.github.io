---
layout: post
title: Ruby - Troubleshooting
date: 2017-12-26 19:23:05 +0300
access: public
comments: true
categories: [ruby]
---

<!-- more -->

* TOC
{:toc}
<hr>

Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
------------------------------------------------------------------

the error occurred after installing new Ruby version (2.5.0):

```
$ bundle
...
An error occurred while installing json (1.8.3), and Bundler cannot continue.
Make sure that `gem install json -v '1.8.3'` succeeds before bundling.
```

**solution**

try to update offending gem:

```sh
$ bundle update json
```

uninitialized constant Gem::BundlerVersionFinder (NameError)
------------------------------------------------------------

```
$ berks
...
/Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems/dependency.rb:283:in `matching_specs': uninitialized constant Gem::BundlerVersionFinder (NameError)
```

**solution**

error was gone after upgrading Ruby version from 2.5.0 to 2.5.1:

```
$ echo 2.5.1 > .ruby-version
```

if this doesn't help, try to clean up old versions of installed gems
(this will remove currently installed versions as well):

```sh
$ gem cleanup
$ bundle
```

`gem cleanup` removes bundled gems only (gems specified in _Gemfile_ of
current project) though it might affect other projects if they are using
the same gems (run `bundle` for those projects as well in that case).

***UPDATE***

it has turned out the problem was caused by specific version of either
`berkshelf` or `chef` or both.

old versions (there is error):

```ruby
gem 'berkshelf', '6.3.1'
gem 'chef', '13.5.3'
```

old versions (error is gone):

```ruby
gem 'berkshelf', '7.0.2'
gem 'chef', '14.1.1'
```
