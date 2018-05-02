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
/Users/tap/.rbenv/versions/2.5.0/lib/ruby/2.5.0/rubygems/dependency.rb:283:in
`matching_specs': uninitialized constant Gem::BundlerVersionFinder (NameError)
```

**solution**

error was gone after upgrading Ruby version from 2.5.0 to 2.5.1:

```
$ echo 2.5.1 > .ruby-version
```
