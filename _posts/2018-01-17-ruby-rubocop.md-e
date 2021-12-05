---
layout: post
title: Ruby - Rubocop
date: 2018-01-17 17:33:15 +0300
access: public
comments: true
categories: [ruby, rubocop]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

usage
-----

### ignore Rubocop errors on a one-off basis

1. <https://github.com/bbatsov/rubocop/blob/master/manual/configuration.md#disabling-cops-within-source-code>

```ruby
# rubocop:disable all
# rubocop:disable Metrics/LineLength, Style/StringLiterals
[...]
# rubocop:enable Metrics/LineLength, Style/StringLiterals
# rubocop:enable all

for x in (0..19) # rubocop:disable Style/For
```

it's required to re-enable checks or else Rubocop will issue warning
that would remind you to do that.

### run Rubocop

```sh
$ rubocop
```
