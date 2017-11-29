---
layout: post
title: Chrome - Tips
date: 2017-01-18 14:47:40 +0300
access: public
comments: true
categories: [chrome]
---

<!-- more -->

## [how to] delete autosuggested URL

`<S-Del>`

## web inspector

### [how to] find elements in Console

- by css: `$('div.phone')`
- by xpath: `$x("//div[contains(@class,'phone')")`

to be able to reveal found elements in Elements Panel
it's necessary to select particular element from collection:

```javascript
$('div.phone')[0]
```