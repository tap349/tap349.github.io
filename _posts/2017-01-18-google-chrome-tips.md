---
layout: post
title: Google Chrome - Tips
date: 2017-01-18 14:47:40 +0300
access: public
comments: true
categories: [chrome]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) delete autosuggested URL
---------------------------------

`<S-Del>`

web inspector
-------------

### (how to) find elements in Console

- by css: `$('div.phone')`
- by xpath: `$x("//div[contains(@class,'phone')")`

to be able to reveal found elements in Elements Panel
it's necessary to select particular element from collection:

```javascript
$('div.phone')[0]
```

(how to) disable material design bookmarks
------------------------------------------

[chrome://flags/#enable-md-bookmarks]() -> `Disabled`

***UPDATE***

this flag is no longer available in Google Chrome 69 (or maybe even 68).

(how to) return old-style tabs
------------------------------

<chrome://flags/#top-chrome-md> -> `Normal`

(how to) copy percent-encoded URL
---------------------------------

1. <https://stackoverflow.com/a/34634019/3632318>

add space to the end of URL and copy.
