---
layout: post
title: Elasticsearch
date: 2019-01-17 15:04:17 +0300
access: public
comments: true
categories: [elasticsearch]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://habr.com/ru/post/280488> (in Russian)

Search (Search APIs)
--------------------

1. <https://www.elastic.co/guide/en/elasticsearch/guide/master/language-intro.html>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html>
>
> The search API allows you to execute a search query and get back search hits
> that match the query.

### `term` query vs. `match` query

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html>
>
> The term query looks for the exact term in the field's inverted index — it
> doesn’t know anything about the field's analyzer. This makes it useful for
> looking up values in keyword fields, or in numeric or date fields. When
> querying full text fields, use the match query instead, which understands
> how the field has been analyzed.

=> `term` for queries is like `keyword` for string field datatypes.

### `most_fields` query type

1. <https://www.elastic.co/guide/en/elasticsearch/guide/master/most-fields.html>

it's used to combine the scores from all matching fields.

Analyze (Indices APIs)
----------------------

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html>
>
> Performs the analysis process on a text and return the tokens breakdown of the text.

### language analyzers

1. <https://www.elastic.co/guide/en/elasticsearch/guide/master/language-intro.html>
2. <https://github.com/C******d/erebor/blob/master/app/chewy/products_index.rb>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html>
>
> The default standard analyzer drops most punctuation, breaks up text into
> individual words, and lower cases them. For instance, the standard analyzer
> would turn the string "Quick Brown Fox!" into the terms [quick, brown, fox].

### testing analyzers

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/_testing_analyzers.html>
1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html>
>
> [Analyze] Can be used without specifying an index against one of the many
> built in analyzers.

say, using cURL:

```sh
$ curl \
  -X GET "http://username:password@localhost:9200/_analyze?pretty" \
  -H "Content-Type: application/json" \
  -d '{"analyzer":"standard","text":"apel"}'

$ curl \
  -X GET "http://username:password@localhost:9200/_analyze?pretty" \
  -H "Content-Type: application/json" \
  -d '{"analyzer":"indonesian","text":"apel"}'
```

### multi-fields

1. <https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html>
2. <https://www.elastic.co/guide/en/elasticsearch/guide/master/multi-fields.html>
3. <https://www.elastic.co/guide/en/elasticsearch/guide/master/most-fields.html>

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html#_multi_fields>
>
> It is often useful to index the same field in different ways for different
> purposes. This is the purpose of multi-fields. For instance, a string field
> could be mapped as a text field for full-text search, and as a keyword field
> for sorting or aggregations. Alternatively, you could index a text field with
> the standard analyzer, the english analyzer, and the french analyzer.
>
> This is the purpose of multi-fields. Most datatypes support multi-fields via
> the fields parameter.

```
{
  "mappings": {
    "_doc": {
      "properties": {
        "description": {
          "type": "keyword",
          "fields": {
            "text": {
              "type": "text",
              "analyzer": "indonesian"
            }
          }
        }
      }
    }
  }
}
```

now it's possible to use both fields in the samy query:

```
GET /_search
{
  "query": {
    "multi_match": {
      "type": "most_fields",
      "query": "barang",
      "fields": ["description", "description.text"]
    }
  }
}
```

> <https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html>
>
> Multi-fields do not change the original _source field.

=> `_source` field contains only original indexed fields - there are no
multi-fields and `text` values (if any) are not analyzed:

```
GET /_search
{
  "size": 100,
  "_source": ["description", "description.text"]
}
```

=>

```json
{
  "took": 18,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 38999,
    "max_score": 1.0,
    "hits": [
      {
        "_index": "reika_shops-1547776421298845",
        "_type": "_doc",
        "_id": "37137675",
        "_score": 1.0,
        "_source": {
          "description": "Online M"
        }
      }
    ]
  }
}
```
