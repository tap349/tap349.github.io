---
layout: post
title: React Native - Flexbox
date: 2017-11-19 00:01:29 +0300
access: public
comments: true
categories: [react-native, flexbox]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://css-tricks.com/snippets/css/a-guide-to-flexbox/>
2. <https://facebook.github.io/react-native/docs/layout-props.html#flex>

all RN components implicitly have `display: flex`.

flex items (children) are placed inside flex container (parent).

### box-sizing

flexbox in RN doesn't have `box-sizing` property but flex items behave
as if they have `box-sizing: content-box` (not `border-box` - that is
padding and border are not included in component's width and height).

### flex item properties

- `flex`

  - `0` - component is inflexible, sized according to its `width` and `height`
  - `-1` - component is inflexible, sized according to its `width` and `height`,
    can be shrinked to its `minWidth` and `minHeight`
  - `1` - component is flexible, sized proportional to specified flex value
    (can be \> 1)
