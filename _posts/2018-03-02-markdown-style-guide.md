---
layout: post
title: Markdown - Style Guide
date: 2018-03-02 21:46:06 +0300
access: public
comments: true
categories: [markdown]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

### use blockquotes for quotations and current page titles (say, wizard steps)

> 1. Step 1: Create Policy

NOTE: backticks are not used.

### use table to specify context and path

1. <https://kramdown.gettalong.org/syntax.html#tables>

[1]: {% post_url 2019-07-19-prettier %}

<dl>
  <dt>context</dt>
  <dd>service, web page, program, etc.</dd>

  <dt>path</dt>
  <dd>location within context</dd>
</dl>

- context is specified in a table header, paths are specified in table rows:

  | iTunes Connect                            |
  | ----------------------------------------- |
  | `My Apps` → `<MY_APP>` → `Features` (tab) |

- context only:

  | iTunes Connect |
  | -------------- |
  |                |

- path only (prefer to always specify context though):

  |                                           |
  | ----------------------------------------- |
  | `My Apps` → `<MY_APP>` → `Features` (tab) |

=> always add header and at least one row - even if they are empty (otherwise
Prettier breaks formatting of Markdown tables - see [Prettier][1]).

it's possible to add path element description in parentheses: usually it's UI
element which represents path element itself (say, menu item or button) or UI
element containing described path element (say, top menu).

path might include action or setting as a final node (say, menu item). if there
are multiple settings, they are specified separately as a list:

| iTunes Connect                    |
| --------------------------------- |
| `Preferences` → `Keys` → `Hotkey` |

- [x] `Show/hide iTerm2 with a system-wide hotkey`
- `Hotkey`: `<C-D-t>`

NOTE: backticks are used for exact names.

keep path on a single line (or else table won't be rendered) but it's allowed to
use multiple table rows each row containing the path on a separate screen
(window or page). or else multiple table rows can be used to split path into 2
parts - high-level part and the one specifying exact location of UI element.

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
italics (_yarn.lock_) where possible - probably except for logs (there are no
comments in logs).

### insert resource link followed by a blank line inside quotes

> <https://rossta.net/blog/from-sprockets-to-webpack.html#deploying-with-capistrano-and-nginx>
>
> set public/packs and node_modules as shared directories to ensure Webpack
> build output and NPM package installation via Yarn are shared across deploys

in some cases when all quotes within section have the same source (and its link
is given at the beginning of the section), the link inside quotes can be
omitted.

### insert resource links at the beginning of the section

1. <https://stackoverflow.com/a/32950454/3632318>

insert resource links if linked resource is related to section subject and can
be useful in one way or another - it doesn't have to be quoted inside this
section or treat the subject directly.

if link is already used inside quotes and information from linked resource is
not used outside these quotes, the link can be omitted at the beginning of the
section.

### use backticks for shell commands but not for proper names

`psql` (shell command) vs. PostgreSQL (proper name).

### try to minimize usage of backticks in headers

say, it's okay to use `psql` in main text but it's better to omit backticks when
it's used in any header.

still in many cases it's necessary to use backticks to avoid ambiguity (say, for
shell commands with arguments - that is when they contain spaces).
