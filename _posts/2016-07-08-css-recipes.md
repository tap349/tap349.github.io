---
layout: post
title: CSS recipes
date: 2016-07-08 02:06:46 +0300
access: public
comments: true
categories: [css]
---

CSS recipes.

<!-- more -->

* TOC
{:toc}
<hr>

## using icons

### global classes (text element is next to icon div)

this method doesn't allow to have multiline text next to icon:
icon and text divs are located on the same line -
text will wrap to the next line below the line with icon div.

_app/assets/stylesheets/globals/icons.sass_:

```sass
@import v2/globals/variables
@import v2/mixins/retina

.success-icon
  +background_inline_retina('#{$image_global_path}/success', 'png', $w: 15px, $h: 16px)
  display: inline-block
  height: 16px
  margin-right: 5px
  vertical-align: baseline
  width: 15px
```

USAGE:

- _app/views/sites/show.html.slim_:

  ```slim
  .success
    .success-icon
    = t '.success_text'
  ```

### global classes (text element is nested in icon div)

this method allows to have multiline text next to icon:
text element is located in icon div (div with icon as background).

_app/assets/stylesheets/globals/icons.sass_:

```sass
@import v2/globals/variables
@import v2/mixins/retina

.success-icon
  +background_inline_retina('#{$image_global_path}/success', 'png', $w: 15px, $h: 16px)
  display: inline-block
  min-height: 16px
  padding-left: 20px
```

USAGE:

- _app/views/sites/show.html.slim_:

  ```slim
  .success-icon
    = t '.success_text'
  ```

even though this method allows for miltiline text html is a bit misleading:
`.success-icon` div contains not only icon but text as well.

### mixin with before pseudo-element

this method doesn't allow to have multiline text next to icon.

_app/assets/stylesheets/mixins/before_icon.sass_:

```sass
@import v2/mixins/retina

=before_icon($image_path, $w: auto, $h: auto, $position: 0 0, $vertical_align: text-bottom)
  &::before
    +background_inline_retina($image_path, 'png', $w: $w, $h: $h, $position: $position)
    content: ''
    display: inline-block
    height: $h
    margin-right: 5px
    vertical-align: $vertical_align
    width: $w
```

USAGE:

- _app/views/sites/show.html.slim_:

  ```slim
  .success
    = t '.success_text'
  ```

- _app/assets/stylesheets/pages/sites.sass_:

  ```sass
  .success
    +before_icon('#{$image_global_path}/success', $w: 16px, $h: 16px)
  ```
