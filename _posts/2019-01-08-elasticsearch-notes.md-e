---
layout: post
title: Elasticsearch - Notes
date: 2019-01-08 22:17:01 +0300
access: public
comments: true
categories: [elasticsearch]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

GET vs. POST
------------

1. <https://www.elastic.co/guide/en/elasticsearch/guide/current/_empty_search.html>

> <https://stackoverflow.com/questions/978061/983458#comment53906725_983458>
>
> Elasticsearch is a fairly major product that utilises HTTP request bodies
> in GET. According to their manual whether a HTTP request should support
> having a body or not is undefined. I'm personally not comfortable with
> populating a GET request body, but they seem to have a different opinion
> and they must know what they're doing.

> <https://stackoverflow.com/a/36940185>
>
> Although the RFC does not mention that you can't have a body with GET.
> So Elasticsearch supports also POST.

- it's possible to use both GET and POST requests for queries which must be
  GET requests semantically (say, search requests)
- it's possible to pass all parameters in JSON request body in both GET and
  POST requests

index
-----

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html>

it's not necessary to create index separately - it's created automatically
when the first document is added to this index.

type
----

> <https://www.elastic.co/guide/en/elasticsearch/reference/6.x/removal-of-types.html#_schedule_for_removal_of_mapping_types>
>
> Indices created in 6.x only allow a single-type per index. Any name can
> be used for the type, but there can be only one. The preferred type name
> is _doc, so that index APIs have the same path as they will have in 7.0:
> PUT {index}/_doc/{id} and POST {index}/_doc.

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
