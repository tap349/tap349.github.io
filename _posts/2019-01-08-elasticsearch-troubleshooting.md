---
layout: post
title: Elasticsearch - Troubleshooting
date: 2019-01-08 22:17:46 +0300
access: public
comments: true
categories: [elasticsearch]
---

<!-- more -->

* TOC
{:toc}
<hr>

NOTE: many problems with Elasticsearch are caused by running out of memory.

Document mapping type name can't start with '_'
-----------------------------------------------

1. <https://discuss.elastic.co/t/cant-use-doc-as-type-despite-it-being-declared-the-preferred-method/113837>

**solution**

even though `_doc` type is recommended as a default type in Elasticsearch 6.x,
leading underscore in type names can be used since Elasticsearch 6.2.0 only:

> <https://discuss.elastic.co/t/cant-use-doc-as-type-despite-it-being-declared-the-preferred-method/113837/9>
>
> Sorry for the confusion, we addressed this in 6.2.0.

"type":"mapper_parsing_exception","reason":"failed to parse [extra_data.goal]"
------------------------------------------------------------------------------

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

[o.e.x.m.j.p.NativeController] Native controller process has stopped - no new native processes can be started
-------------------------------------------------------------------------------------------------------------

**solution**

1. <https://github.com/elastic/elasticsearch/issues/25067>
2. <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html>

try to increase memory limits:

```diff
  services:
    elasticsearch:
      environment:
-       ES_JAVA_OPTS: -Xms256m -Xmx256m
+       ES_JAVA_OPTS: -Xms512m -Xmx512m
-     mem_limit: 500m
+     mem_limit: 1g
```
