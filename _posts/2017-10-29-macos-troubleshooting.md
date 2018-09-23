---
layout: post
title: macOS - Troubleshooting
date: 2017-10-29 01:42:13 +0300
access: public
comments: true
categories: [macos]
---

<!-- more -->

* TOC
{:toc}
<hr>

battery percentage is not properly updating
-------------------------------------------

1. <https://support.apple.com/en-us/HT201295>

reset the SMC.

active developer directory is a CLT instance
--------------------------------------------

> xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer
> directory '/Library/Developer/CommandLineTools' is a command line tools instance

1. <http://stackoverflow.com/questions/17980759#17980786>

just switch to Xcode directory:

```sh
$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```

unable to update Xcode from 8.2.1 on El Capitan
-----------------------------------------------

1. <https://stackoverflow.com/questions/43065475/unable-to-update-to-xcode-8-3>

Xcode 8.3 and higher requires macOS Sierra.

very slow paste of text into any terminal
-----------------------------------------

the problem appeared after upgrading macOS from El Capitan to High Sierra:
when text is pasted into the terminal, there's a significant delay (it's
proportional to the length of text but, say, for 80 characters it might be
about 1 second) and then pasted text is automatically selected.

**solution**

1. <https://cirw.in/blog/bracketed-paste>
2. <http://zsh.sourceforge.net/Doc/Release/Parameters.html>

it has turned out that this behaviour is called `bracketed paste mode`
and is meant to protect you from malevolent input: when bracketed paste
mode is disabled, pasted text is immediately executed but after turning
this mode on you have an opportunity to examine what you are going to
paste and then press `<CR>` to continue with pasting.

bracketed paste mode is supported by both iTerm2 and Zsh so it can be
turned off on any level (I'm turning it off because I'm not used to it
and because it introduces significant and annoying delay when pasting):

- iTerm2

  1. <https://gitlab.com/gnachman/iterm2/issues/5698#note_27701613>

  | `Preferences` → `Profiles` → `Default` → `Keys`

  map `<D-v>` keyboard shortcut to `Paste..` action and
  disable `Bracketed paste mode` in action settings.

- Zsh

  1. <https://stackoverflow.com/a/33456330/3632318>

  _~/.zshrc_:

  ```zsh
  unset zle_bracketed_paste
  ```

tab navigation doesn't work
---------------------------

OS-wide tab navigation shorcuts (`<D-S-[>`, `<D-S-[>`) no longer work
after upgrading:

- Google Chrome: → 69.0.3497.100
- macOS: 10.13.5 → 10.13.6

**solution**

1. <https://superuser.com/a/1260437/326775>

| Preferences: Keyboard → Shortcuts (tab) → App Shorcuts

add these shortcuts:

- Google Chrome:
  - Select Previous Tab: `<D-S-[>`
  - Select Next Tab: `<D-S-[>`

or else use some Chrome extension like Vimium or cVim.
