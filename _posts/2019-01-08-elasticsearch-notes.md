---
layout: post
title: Elasticsearch - Notes
date: 2019-01-08 22:17:01 +0300
access: public
comments: true
categories: [elasticsearch]
---

<!-- more -->

* TOC
{:toc}
<hr>

index
-----

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html>

it's not necessary to create index separately - it's created automatically
when the first document is added to this index.

document
--------

> <https://www.elastic.co/guide/en/elasticsearch/guide/2.x/document.html>
>
> In Elasticsearch, the term document has a specific meaning. It refers to
> the top-level, or root object that is serialized into JSON and stored in
> Elasticsearch under a unique ID.

### settings

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

### mapping

1. <https://www.elastic.co/blog/found-elasticsearch-mapping-introduction>

> If you do not specify a mapping, Elasticsearch will by default generate one
> dynamically when detecting new fields in documents during indexing. However,
> this dynamic mapping generation comes with a few caveats.
>
> 1. Detected types might not be correct.
> 2. May lead to unnecessary duplication. (The _source field and _all field especially)
> 3. Uses default analyzers and settings for indexing and searching.

=> specify mapping explicitly.

type
----

> <https://www.elastic.co/guide/en/elasticsearch/reference/6.x/removal-of-types.html#_schedule_for_removal_of_mapping_types>
>
> Indices created in 6.x only allow a single-type per index. Any name can
> be used for the type, but there can be only one. The preferred type name
> is _doc, so that index APIs have the same path as they will have in 7.0:
> PUT {index}/_doc/{id} and POST {index}/_doc.

language analyzers
------------------

1. <https://www.elastic.co/guide/en/elasticsearch/guide/master/language-intro.html>
2. <https://habr.com/ru/post/280488> (in Russian)
3. <https://github.com/C******d/erebor/blob/master/app/chewy/products_index.rb>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html>
>
> The default standard analyzer drops most punctuation, breaks up text into
> individual words, and lower cases them. For instance, the standard analyzer
> would turn the string “Quick Brown Fox!” into the terms [quick, brown, fox].

### `most_fields` query type

1. <https://www.elastic.co/guide/en/elasticsearch/guide/master/most-fields.html>

it's used to combine the scores from all matching fields.

### `term` query vs. `match` query

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html>
>
> The term query looks for the exact term in the field’s inverted index — it
> doesn’t know anything about the field’s analyzer. This makes it useful for
> looking up values in keyword fields, or in numeric or date fields. When
> querying full text fields, use the match query instead, which understands
> how the field has been analyzed.

=> `term` for queries is like `keyword` for string field datatypes.
