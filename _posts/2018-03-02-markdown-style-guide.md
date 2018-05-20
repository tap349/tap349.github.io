---
layout: post
title: Markdown - Style Guide
date: 2018-03-02 21:46:06 +0300
access: public
comments: true
categories: [markdown]
---

<!-- more -->

* TOC
{:toc}
<hr>

- use blockquotes for quotations and current page titles (say, wizard steps):

  > 1) Step 1: Create Policy

  NOTE: backticks are not used.

- use table row to specify path (location within a web page, program, etc.):

  | iTunes Connect: `My Apps` → `<my_app>` → `Features` (tab)

  environment (web page, program, etc.) can be specified before the path.

  path might include action or setting as a final node (say, menu item).
  if there are multiple settings they are specified separately as a list:

  | `Preferences` → `Keys` → `Hotkey`

  - [x] `Show/hide iTerm2 with a system-wide hotkey`
  - `Hotkey`: `<C-D-t>`

  NOTE: backticks are used for exact names.

  keep path on a single line (or else table won't be rendered) but it's
  allowed to use multiple table rows each row containing the path on a
  separate screen (window or page). or else it's multiple table rows can
  be used to split path into 2 parts - high-level part and the one that
  specifies exact location of UI element).

- use comment with arrow to specify return value

  ```elixir
  ~N[2019-09-01 03:00:00]
  |> Timex.shift(months: 1)
  # => ~N[2019-10-01 03:00:00]
  ```

  use this format instead of `pry>` or `iex>` prompts where possible.

- use comment line followed by blank line to specify file path

  1. <https://sass-lang.com/guide>
  2. <https://rossta.net/blog/from-sprockets-to-webpack.html>

  ```javascript
  // yarn.lock

  "@rails/webpacker@3.5":
    version "3.5.3"
  ```

  use this format instead of specifying file path in front of cobe block
  in italics (_yarn.lock_) where possible.
