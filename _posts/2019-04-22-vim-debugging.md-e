---
layout: post
title: Vim - Debugging
date: 2019-04-22 17:47:41 +0300
access: public
comments: true
categories: [vim]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://github.com/c9s/vim-dev-plugin>

current mapping of any key or key combination
---------------------------------------------

- `:verbose map <S-CR>`
- `:verbose nmap ,f`

current option value
--------------------

for `path` option:

- `:set path?`
- `echo &path`

all autocommands currently associated with augroup
--------------------------------------------------

```vim
:autocmd <augroup_name>
```

it might be useful to check there are no duplicate autocommands.

Vim profiling
-------------

1. <https://www.reddit.com/r/vim/comments/2lw1fd/pretty_statuslines_vs_cursor_speed/cm18qgt/>:

> 1. Execute `:profile start result | profile func *`
> 2. Do some stuff (moving cursor around or splitting windows)
> 3. Quit Vim
> 4. Open result file and go to the last part of the log, see what's causing your editor heavy.

print messages about what Vim is doing
--------------------------------------

1. <https://superuser.com/a/747405/326775>
2. `:help 'verbose'`

say, to print every executed autocommand:

```vim
:set verbose=9
```

to print everything:

```vim
:set verbose=15
```

it's possible to set `verbose` level and run a command in one
go in order to understand what's going on when command is run.

e.g., setting `verbose` level right before opening a file can
be used to find out the reason why syntax highlighting is lost:

```vim
:set verbose=15 | edit foo.js
```

print debugging messages
------------------------

when debugging a plugin, messages printed inside the plugin with
`echom[sg]` are not saved in message history (`:messages`) - use
`echoe[rr]` instead.
