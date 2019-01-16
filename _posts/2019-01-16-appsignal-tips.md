---
layout: post
title: AppSignal - Tips
date: 2019-01-16 11:50:55 +0300
access: public
comments: true
categories: [appsignal]
---

<!-- more -->

* TOC
{:toc}
<hr>

(how to) notify AppSignal of a new deploy
-----------------------------------------

1. <https://docs.appsignal.com/application/markers/deploy-markers.html#config-option>

add `revision` config option:

```diff
  # config/appsignal.exs

  config :appsignal, :config,
    active: true,
-   name: "MyApp"
+   name: "MyApp",
+   revision: Mix.Project.config()[:version],
```

NOTE: don't forget to update project version in _mix.exs_.
