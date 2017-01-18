---
layout: post
title: Chrome web inspector tips
date: 2017-01-18 14:47:40 +0300
access: public
categories: [chrome]
---

Chrome web inspector tips.

<!-- more -->

## find elements in Console

- by css: `$('div.phone')`
- by xpath: `$x("//div[contains(@class,'phone')")`

to be able to reveal found elements in Elements Panel
it's necessary to select particular element from collection:

```javascript
$('div.phone')[0]
```
