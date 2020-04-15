---
layout: post
title: React - Material UI
date: 2020-04-15 13:28:17 +0300
access: public
comments: true
categories: [react, mui]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## spacing

1. <https://material-ui.com/system/spacing/>

- use `Box` component for outer spacing (`margin`, `padding`) where possible

  in some cases additional `Box` components may break existing flexbox layout
  because by default they have `display: block` property.

  in this case to avoid adding extra styles (say, `display: flex; flex: 1` to
  `Box` and `flex: 1` to its child) remove `Box` wrapper at all and add outer
  spacing properties directly to child component styles.

  in simple cases and unless required still prefer `Box` to layout components.

- use styles for inner styling (including width and height)
