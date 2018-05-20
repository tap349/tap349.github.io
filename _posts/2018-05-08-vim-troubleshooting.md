---
layout: post
title: Vim - Troubleshooting
date: 2018-05-08 12:46:55 +0300
access: public
comments: true
categories: [vim]
---

<!-- more -->

* TOC
{:toc}
<hr>

syntastic
---------

in my case problem was with invalid _.rubocop.yml_ file. this can be easily
checked by running `rubocop` in the project root: if rubocop starts analyzing
your project then all is fine otherwise it complains about invalid syntax -
in that case just follow on-screen instructions and fix _.rubocop.yml_.

to debug inside Vim turn on syntastic debugging messages in command-line mode:

```vim
:let g:syntastic_debug = 1 " or 3 or 33
```

run syntastic checkers:

```vim
:SyntasticCheck mri rubocop
```

view syntastic debugging messages in Vim messages:

```vim
:messages
```

in my case rubocop returned status code 2 which means running rubocop checker
failed on rubocop side.

BTW always set path to ruby and rubocop executables manually so as not to
rely on correct `PATH` inside Vim (it can be messed up for a million reasons):

```vim
let g:syntastic_ruby_mri_exec = '~/.rbenv/shims/ruby'
let g:syntastic_ruby_rubocop_exec = '~/.rbenv/shims/rubocop'
```

[MacVim] blank screen on macOS Sierra and above
-----------------------------------------------

the problem didn't exist in El Capitan - it first appeared in Sierra:
the whole screen went blank when starting to move a cursor around.

**solution**

1. <https://github.com/macvim-dev/macvim/wiki/FAQ#black-screen-on-full-screen>
2. <https://github.com/macvim-dev/macvim/issues/557>
3. <https://github.com/macvim-dev/macvim/commit/b115d75a66847c4a49770161b105a9a64685c043>

```zsh
# ~/.zlogin

defaults write org.vim.MacVim MMUseCGLayerAlways 1
```

iTerm2 and MacVim hang when running ALE
---------------------------------------

this happens only when file is saved several times in quick succession
AND `htop` is opened in iTerm2 (or another terminal) - it freezes both
iTerm2 and MacVim somehow.

**solution**

this is a known bug in `htop`, macOS High Sierra or both:
<https://github.com/hishamhm/htop/issues/682>.

***UPDATE 2018-02-26***

`htop` 2.0.2 is back in brew even though the issue is still open.

location list becomes blank after opening Markdown file
-------------------------------------------------------

this happens when trying to open some Markdown files from location
list with search results.

**solution**

the problem is not Markdown-specific and is caused by ALE linting first
opened files (`g:ale_lint_on_enter` option is enabled by default): after
running linters ALE populates location list of current window with found
issues (or just clears it if no issues are found) - anyway the previous
contents of location list (search results) are gone.

to circumvent this problem it's possible:

- to disable linting of first opened files
  (`let g:ale_lint_on_enter = 0`)
- to disable populating location list with issues found by ALE
  (`let g:ale_set_loclist = 0`)

when using mouse, cursor is positioned with noticeable lag
----------------------------------------------------------

this might be caused by source code tooltips (balloons) - disable them:

```vim
" ~/.vim/vimrc

set noballooneval
```

for Ruby it's also necessary to unset `balloonexpr` ftplugin file:

```vim
" ftplugin/ruby.vim

setlocal balloonexpr=
```

creating alternate file doesn't work
------------------------------------

after running `:AV` command (same as `:confirm A`) there is no prompt
to create alternate file if it's missing:

```
E345: Can't find file "/Users/tap/dev/sith/spec/controllers/webhooks_controller_spec.rb" in path
```

**solution**

1. <https://github.com/tpope/vim-rails/issues/503#issuecomment-387344958>

I created new Rails project with `--no-skip-test` => `test` directory
was created instead of `spec` one and `vim-rails` plugin couldn't find
or create alternate file. so just rename directory to fix the issue.

incorrect folding of lines with equal indent
--------------------------------------------

```vim
" ~/.vim/vimrc

set foldmethod=indent
```

lines with equal indent don't form a fold - folding stops at blank line.

**solution**

comment to <https://stackoverflow.com/a/14803964/3632318>:

> If I've reproduced the problem and run :set shiftwidth=2 after the text has
> already been input, it works as expected but not if i do :set shiftwidth=2
> before the text has been input. Similarly if the file has been saved and
> reopened, the shiftwidth=2 in .vimrc works and the fold works as expected.

all in all if folding is somehow broken, just reload file with `:e` command.
