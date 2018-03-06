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

  NOTE: keep path on a single line (or else table won't be rendered) though
        it's possible to have 2 table rows (the 1st one for high-level path
        and the 2nd one - to specify exact location of UI element):

  | iTunes Connect: `My Apps` → `Users and Roles`
  | `Sandbox Testers` (tab) → `Testers ⨁`

  try not to overuse it for hotkeys and menu selections.

- when path is inlined (say, element of the list), don't wrap it in table:

  - Xcode: `Product` (top menu) → `Destination` → `Add Additional Simulators...`
  - press `+` icon (`Add Simulator`) in bottom left corner
