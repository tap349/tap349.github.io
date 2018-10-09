---
layout: post
title: Safari - Troubleshooting
date: 2018-10-09 23:12:44 +0300
access: public
comments: true
categories: [safari]
---

<!-- more -->

* TOC
{:toc}
<hr>

can't access ~/Library/Safari/
------------------------------

I can neither cd to this directory nor list its files even though I am the
owner and permissions seem to be okay (755) - I keep on getting `Operation
not permitted` error even when using `sudo`.

**solution**

for some strange reason this directory can be accessed with Finder app only.

history entry can't be deleted
------------------------------

when I press `Del` button, the next history entry is selected without deleting
current one. clicking `Delete` menu item in context menu has no effect as well.

**solution**

1. <https://apple.stackexchange.com/questions/316943/cant-delete-safari-history-entry-on-mac>

this happened right after I imported history from Google Chrome so the first
tip is to avoid importing history (bookmarks were imported without a hitch).

if you already have the problem, it might help to delete _History.db*_ files
in _~/Library/Safari/_ directory and restart the browser (or even OS) - maybe
they got corrupted while importing history.

also you might try to remove _~/Library/Caches/com.apple.Safari/_ directory.

Safari doesn't load the page after entering URL
-----------------------------------------------

see the previous issue - symptoms and solution are the same.
