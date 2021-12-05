---
layout: post
title: Git - Troubleshooting
date: 2019-06-13 13:40:59 +0300
access: public
comments: true
categories: [git]
---

<!-- more -->

* TOC
{:toc}
<hr>

'upstream' does not appear to be a git repository
-------------------------------------------------

```sh
$ git fetch upstream
fatal: 'upstream' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

**solution**

1. <https://github.com/Esri/developer-support/wiki/Setting-the-upstream-for-a-fork>

```sh
$ git remote add upstream <UPSTREAM_REPO>
$ git remote -v
```
