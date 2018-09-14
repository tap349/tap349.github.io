---
layout: post
title: Elasticsearch
date: 2018-01-18 17:43:03 +0300
access: public
comments: true
categories: [elasticsearch, rails]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model>
2. <https://www.elastic.co/guide/en/elasticsearch/guide/2.x/document.html>

- models inside Elasticsearch are called documents
- documents in Elasticsearch are identified by model ids

import all `ProductEvent` models into Elasticsearch
(previous imported models will be replaced):

```sh
$ ProductEvent.__elasticsearch__.import
```

sync specific model (`POST` request will be issued
if model has been indexed, `PUT` request otherwise):

```sh
$ ProductEvent.last.__elasticsearch__.index_document
```

troubleshooting
---------------

### "type":"mapper_parsing_exception","reason":"failed to parse [extra_data.goal]"

```
pry> Api::Elastic::Index.call({ id: 1, extra_data: { 'goal' => 'reach' } })
< {"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":"failed
to parse [extra_data.goal]"}],"type":"mapper_parsing_exception","reason":"failed
to parse [extra_data.goal]","caused_by":{"type":"illegal_argument_exception",
"reason":"For input string: \"likes\""}},"status":400}

[400] {"error":{"root_cause":[{"type":"mapper_parsing_exception","reason":
"failed to parse [extra_data.goal]"}],"type":"mapper_parsing_exception","reason":
"failed to parse [extra_data.goal]","caused_by":{"type":"illegal_argument_exception",
"reason":"For input string: \"likes\""}},"status":400}

Elasticsearch::Transport::Transport::Errors::BadRequest: [400] {"error":{
"root_cause":[{"type":"mapper_parsing_exception","reason":"failed to parse
[extra_data.goal]"}],"type":"mapper_parsing_exception","reason":"failed to
parse [extra_data.goal]","caused_by":{"type":"illegal_argument_exception",
"reason":"For input string: \"likes\""}},"status":400}
```

**solution**

it turns Elasticsearch 6.1 doesn't allow to have `goal` attribute while
Elasticsearch 6.2 allows. so solution is to either rename the attribute
or upgrade Elasticsearch.
