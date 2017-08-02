---
layout: post
title: Chef - Troubleshooting
date: 2017-08-02 18:25:32 +0300
access: public
categories: [chef]
---

<!-- more -->

* TOC
{:toc}
<hr>

## Doing old-style registration with the validation key at

```
$ knife zero bootstrap builder --node-name builder
Doing old-style registration with the validation key at ...
Delete your validation key in order to use your user credentials instead
```

**solution**

<https://github.com/higanworks/knife-zero/issues/24>

> Don't worry about it.
> This warning message is not effect Knife-Zero's behavior.
> Because authrization of Chef-Zero is dummy.
