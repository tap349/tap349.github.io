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

import all `ProductEvent` models into Elasticsearch (previous imported models
will be replaced):

```sh
$ ProductEvent.__elasticsearch__.import
```

sync specific model (`POST` request will be issued if model has been indexed,
`PUT` request otherwise):

```sh
$ ProductEvent.last.__elasticsearch__.index_document
```

notes
-----

### index

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html>

it's not necessary to create index separately - it's created automatically
when the first document is added to this index.

#### settings

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html>
>
> You do not have to explicitly specify index section inside the settings section.

```json
{
  "settings" : {
    "number_of_shards" : 3,
    "number_of_replicas" : 2
  }
}
```

is equivalent to:

```json
{
  "settings" : {
    "index" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 2
    }
  }
}
```

### type

> <https://www.elastic.co/guide/en/elasticsearch/reference/6.x/removal-of-types.html#_schedule_for_removal_of_mapping_types>
>
> Indices created in 6.x only allow a single-type per index. Any name can
> be used for the type, but there can be only one. The preferred type name
> is _doc, so that index APIs have the same path as they will have in 7.0:
> PUT {index}/_doc/{id} and POST {index}/_doc.

troubleshooting
---------------

### Document mapping type name can't start with '_'

1. <https://discuss.elastic.co/t/cant-use-doc-as-type-despite-it-being-declared-the-preferred-method/113837>

**solution**

even though `_doc` type is recommended as a default type in Elasticsearch 6.x,
leading underscore in type names can be used since Elasticsearch 6.2.0 only:

> <https://discuss.elastic.co/t/cant-use-doc-as-type-despite-it-being-declared-the-preferred-method/113837/9>
>
> Sorry for the confusion, we addressed this in 6.2.0.

### "type":"mapper_parsing_exception","reason":"failed to parse [extra_data.goal]"

```
pry> Api::Elastic::Index.call({ id: 1, extra_data: { 'goal' => 'reach' } })

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
