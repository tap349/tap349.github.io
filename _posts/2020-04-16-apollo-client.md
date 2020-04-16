---
layout: post
title: Apollo Client
date: 2020-04-16 15:22:44 +0300
access: public
comments: true
categories: [react, graphql]
---

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

## troubleshooting

### data returned by useQuery hook is always undefined

query:

```
team(id: $id) @client {
  id
  name
  year
  avatarUrl
  coach {
    id
    name
  }
}
```

resolver function in local resolver map:

```javascript
{
  __typename: 'Team',
  id: 1,
  name: 'Длинное бурундоковое название',
  year: 2012,
  avatarUrl: null,
  coach: {
    __typename: 'User',
    name: 'Константинопольский Константин',
  }
}
```

**solution**

1. <https://github.com/apollographql/apollo-client/issues/4267>

the problem is that client asks for `id` field in `coach` field but it's not
returned by server (local resolver here).

it turns out that Apollo Client silently ignores this error without telling
us that something went wrong => `useQuery` hook returns this result object:

```javascript
{
  loading: false,
  error: undefined,
  data: undefined
}
```

BTW it's okay to return more fields in resolver function than requested.
