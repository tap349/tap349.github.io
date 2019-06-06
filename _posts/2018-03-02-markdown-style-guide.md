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

### use blockquotes for quotations and current page titles (say, wizard steps)

> 1) Step 1: Create Policy

NOTE: backticks are not used.

### use table row to specify path (location within a web page, program, etc.)

| iTunes Connect: `My Apps` → `<MY_APP>` → `Features` (tab)

environment (web page, program, etc.) can be specified before the path.

it's possible to add path element description in parentheses: usually it's UI
element which represents path element itself (say, menu item or button) or UI
element containing described path element (say, top menu).

path might include action or setting as a final node (say, menu item). if there
are multiple settings they are specified separately as a list:

| `Preferences` → `Keys` → `Hotkey`

- [x] `Show/hide iTerm2 with a system-wide hotkey`
- `Hotkey`: `<C-D-t>`

NOTE: backticks are used for exact names.

keep path on a single line (or else table won't be rendered) but it's allowed
to use multiple table rows each row containing the path on a separate screen
(window or page). or else multiple table rows can be used to split path into 2
parts - high-level part and the one that specifies exact location of UI element).

### use comment with arrow to specify return value

```elixir
~N[2019-09-01 03:00:00]
|> Timex.shift(months: 1)
# => ~N[2019-10-01 03:00:00]
```

use this format instead of `pry>` or `iex>` prompts where possible.

### use comment line followed by blank line to specify file path inside code block

1. <https://sass-lang.com/guide>
2. <https://rossta.net/blog/from-sprockets-to-webpack.html>

```javascript
// yarn.lock

"@rails/webpacker@3.5":
  version "3.5.3"
```

use this format instead of specifying file path in front of cobe block in
italics (_yarn.lock_) where possible - probably except for logs (there are
no comments in logs).

### insert link or path to the resource followed by blank line inside blockquotes

> <https://rossta.net/blog/from-sprockets-to-webpack.html#deploying-with-capistrano-and-nginx>
>
> set public/packs and node_modules as shared directories to ensure Webpack
> build output and NPM package installation via Yarn are shared across deploys

in some cases when all quotes within section have the same source (and its
link is given at the beginning of the section) the link inside blockquotes
can be omitted.

### use backticks for shell commands but not for proper names

`psql` (shell command) vs. PostgreSQL (proper name).

### try to minimize usage of backticks in headers

say, it's okay to use `psql` in main text but it's better to omit backticks
when it's used in any header.

still in many cases it's necessary to use backticks to avoid ambiguity (say,
for shell commands with arguments - that is when they contain spaces).
