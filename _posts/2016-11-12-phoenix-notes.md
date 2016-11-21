---
layout: post
title: Phoenix notes
date: 2016-11-12 16:21:40 +0300
access: public
categories: [phoenix, rails]
---

notes about phoenix (maybe this article will be split into several parts later).

<!-- more -->

### IEx

- `iex` - doesn't load your app (equivalent of `irb`)
- `iex -S mix` - loads your app (equivalent of `rails console`)
- `iex -S mix phoenix.server` - loads your app and launches web server
  (`rails console` inside `rails server` - or vice versa)

### associations

associations are always loaded explicitly!

preparation:

```elixir
> alias Rumbl.Repo
> alias Rumbl.User
> import Ecto.Query
```

build and insert association:

```elixir
> attrs = %{title: 'title', description: 'desc'}
> video = Ecto.build_assoc(user, :videos, attrs)
> video = Repo.insert!(video)
```

alternatively build association explicitly:

```elixir
> video = %Video{user_id: user.id, title: 'title', description: 'desc'}
> video = Repo.insert!(video)
```

load association and store it in model struct:

```elixir
> user = Repo.preload(user, :videos)
> videos = user.videos
```

load association without storing it in model struct:

```elixir
> query = Ecto.assoc(user, :videos)
> videos = Repo.all(query)
```
