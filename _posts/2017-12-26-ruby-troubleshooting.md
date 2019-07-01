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

can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
----------------------------------------------------------------------------------

```
$ bundle
Traceback (most recent call last):
	2: from /Users/tap/.rbenv/versions/2.5.0/bin/bundle:23:in `<main>'
	1: from /Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems.rb:308:in `activate_bin_path'
/Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems.rb:289:in `find_spec_for_exe': can't find gem bundler (>= 0.a) with executable bundle (Gem::GemNotFoundException)
```

**solution**

1. <https://stackoverflow.com/a/54038218/3632318>

find current Bundler version:

```
$ gem list --local bundler

*** LOCAL GEMS ***

bundler (2.0.2)
```

find Bundler version in _Gemfile.lock_:

```
# Gemfile.lock

BUNDLED WITH
  1.16.1
```

now there are 2 possible solutions:

### sync Gemfile.lock to Bundler version

```diff
  # Gemfile.lock

  BUNDLED WITH
-   1.16.1
+   2.0.1
```

### sync Bundler version to Gemfile.lock

```sh
$ gem install bundler -v 1.16.1
```

in many cases this solution is preferable since it's easier and causes less
problems with CI and deployment.
