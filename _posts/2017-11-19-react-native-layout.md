---
layout: post
title: React Native - Layout
date: 2017-11-19 00:01:29 +0300
access: public
comments: true
categories: [react-native, flexbox]
---

<!-- more -->

* TOC
{:toc}
<hr>

style guide
-----------

- page should be divided into sections

  sections are `View`s - preferably with the same `marginBottom`
  and (possibly) other properties which can be extracted into local
  `section` style.

- all standard and custom components (buttons, inputs, pickers,
  etc.) must have ZERO margins and external paddings

  if it's necessary to set margins, wrap component into `View` or
  `ScrollView` and style this container with required margins (say,
  using `section` style) - components themselves must be responsible
  for how they look INSIDE only without any assumptions about their
  context.

  consequently, using `containerStyle` component property to set
  margins or external paddings is an antipattern.

flexbox
-------

1. <https://css-tricks.com/snippets/css/a-guide-to-flexbox/>
2. <https://facebook.github.io/react-native/docs/layout-props.html#flex>
3. <https://medium.freecodecamp.org/understanding-flexbox-everything-you-need-to-know-b4013d4dc9af>

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

- `alignItems`

  default value is `stretch` so *items fill all available space along the
  cross-axis by default*!

  if it's necessary to fill all available space along the main axis,
  use `flexGrow: 1` (or more general `flex: 1`).

- `lineHeight`

  when setting `lineHeight`, set `height` to the same value as well:
  say, after setting `lineHeight: 20` the height of `TextInput` is 17.5.

  it's not always possible though: it's impossible to set fixed
  height when text component spans multiple lines. in this case I
  fine-tune `lineHeight` so that (primarily for Android):

  `actual_height` = `number_of_lines` x `original_line_height`

  **UPDATE**

  first of all, get it clear if the problem with invalid text component
  height exists on real Android device before using hacks mentioned above.
