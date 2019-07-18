---
layout: post
title: Elasticsearch - Rails
date: 2018-01-18 17:43:03 +0300
access: public
comments: true
categories: [elasticsearch, rails]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model>

- models inside Elasticsearch are called documents
- documents in Elasticsearch are identified by model IDs

import all `ProductEvent` models into Elasticsearch (previously imported models
will be replaced):

```sh
$ ProductEvent.__elasticsearch__.import
```

sync specific model (`POST` request will be issued if model has been indexed,
`PUT` request otherwise):

```sh
$ ProductEvent.last.__elasticsearch__.index_document
```
