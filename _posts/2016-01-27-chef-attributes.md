---
layout: post
title: Chef - Attributes
date: 2016-01-27 14:06:00 +0300
access: public
comments: true
categories: [chef]
---

guidelines for attributes

<!-- more -->

- cookbooks must not require attributes to be set outside of the cookbook itself to function

  - <http://bytearrays.com/chef-cookbook-patterns/>

- override community cookbook attributes in wrapper cookbooks using `override`

  - <http://bytearrays.com/chef-cookbook-patterns/>

- don't store attributes in roles at all

  - <http://dougireton.com/blog/2013/02/16/chef-cookbook-anti-patterns/>
  - <http://bytearrays.com/chef-cookbook-patterns/>

- store environment-specific attributes in environment files

- don't store attributes in recipes - keep them in attribute files:
  to initialize attributes in environment cookbooks or
  override attributes in wrapper cookbooks

  - <http://deadunicornz.org/blog/2014/06/11/chef-attribute-overrides-in-recipes-is-a-bad-idea>
  - <http://agiletesting.blogspot.ru/2010/11/working-with-chef-attributes.html>

- use only default and override attributes in cookbooks and recipes

  - <https://docs.chef.io/ruby.html>

- don't use local variables inside attribute files - they are eagerly evaluated
  and some node attributes might be not available yet

- prefer strings to symbols as attribute names,
  in any case don't mix different notations

  - <https://docs.chef.io/ruby.html>
