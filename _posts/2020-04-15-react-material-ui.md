---
layout: post
title: React - Material UI
date: 2020-04-15 13:28:17 +0300
access: private
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

- use `Box` component for outer spacing (margin, padding)
- use styles for inner styling (including width and height)

don't use `Box` for inner styling:

> <https://material-ui.com/components/box/#overriding-material-ui-components>
>
> However, sometimes you have to target the underlying DOM element. For
> instance, you want to change the text color of the button. The Button
> component defines its own color. CSS inheritance doesn't help.

To workaround this problem you can use `React.cloneElement()` or `render` props
but in this case you would have to rely on the order of imports which looks
ugly.
