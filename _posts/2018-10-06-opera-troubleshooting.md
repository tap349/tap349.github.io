---
layout: post
title: Opera - Troubleshooting
date: 2018-10-06 13:26:50 +0300
access: public
comments: true
categories: [opera]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

synchronization button is missing
---------------------------------

synchronization button (`person` icon in toolbar next to address bar) is
missing after restarting Opera.

**solution**

> <https://forums.opera.com/topic/24028/synchronization-buttons-missing>
>
> The one at the top right will show up only if needed, to show that
> something requires your attention.
>
> Sync button for a while and if there's any sync problem.
> You have to run sync from the menu every time you have to login.

so it must be okay it's not shown all the time - click this menu item to
force show it:

| Opera: Opera (top menu) → Signed in as \<YOUR_EMAIL>

warning is not shown before quitting with <D-Q>
-----------------------------------------------

setting corresponding option `Show warning before quitting with <D-Q>` on
has no effect - if you close and open settings page again, this option is
reset to off again.

**solution**

set `Warn on closing browser window with multiple tabs` flag to `Enabled`.
