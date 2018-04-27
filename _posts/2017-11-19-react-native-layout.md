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

  sections are `View`s - preferably with the same `paddingBottom`
  and (possibly) other properties which can be extracted into local
  `section` style (if it's used more than once of course).

- all standard and custom components (buttons, inputs, pickers,
  etc.) must have ZERO margins and external paddings by default
  (components themselves must be responsible for how they look
  INSIDE only without any assumptions about their context)

  if it's necessary to set margins, either wrap component into `View`
  and style this container with required margins (say, using `section`
  style) or use `containerStyle` component property (or the like).

  what concerns wrapping ALL standard components (like `Text`) into `View`
  just to add some padding: now I think it's not strictly necessary since
  it introduces a lot of nesting - it's allowed to add layout style props
  right to standard components.

- margin vs. padding

  there are cases when either only padding or only margin would fit -
  say, margin will be used when adding space between child components
  each with its own background color.

  but sometimes it's necessary to add spacing between current and adjacent
  components (usually by wrapping current component into `View` container)
  and both padding and margin would do the job - use padding in this case.

  use padding:

  - for spacing only when component has no its own distinct boundaries
    (border or background color) - it's some kind of container as a rule

  don't use padding:

  - to add spacing via `containerStyle` property of a button since it
    would grow the button instead
  - for components with fixed height or width (vertical padding would "eat"
    component's height and horizontal one would "eat" its width accordingly)

  one advantage of using padding instead of margin is in better support
  of the former by simulator built-in Inspector - for some reason Inspector
  is better at highlighting padding rather than margin.

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

  ***UPDATE***

  get it clear if the problem with invalid text component height exists
  on real Android device before using hacks mentioned above - maybe this
  is an emulator-specific issue.

### flex item and fixed width item in the same row

what we want to get in the end:

```
|John Doe foo foo foo foo foo...  20 y.o.|
|Jane Doe                         30 y.o.|
```

here name item is flexible, age item is fixed. but in case of a long name
(the 1st row) it wouldn't be truncated pushing age item to the right =>
total width of row items becomes larger than the width of flex container:

```
|John Doe foo foo foo foo foo foo   20 y.o.
|Jane Doe                         30 y.o.|
```

we need to shrink flexible item (name item) when row items overflow their
flex container - use `flexShrink: 1` property to achieve this behaviour:

{% raw %}
```jsx
<View style={{flexDirection: 'row'}}>
  <View style={{flex: 1, flexShrink: 1, marginRight: 4}}>
    <Text numberOfLines={1}>{this.props.name}</Text>
  </View>
  <View style={{width: 48}}>
    <Text>{this.props.age}</Text>
  </View>
</View>
```
{% endraw %}
