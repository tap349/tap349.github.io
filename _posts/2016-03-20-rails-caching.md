---
layout: post
title: rails-caching
date: 2016-03-20 16:17:42 +0300
access: public
categories: [rails, caching, turbolinks]
---

notes about caching in rails and turbolinks.

<!-- more -->

* TOC
{:toc}

# cache digests

<https://github.com/rails/cache_digests>

**cache digests** were included in rails 4 and had been available as a gem before.

prior to cache digests it was necessary to expire fragment caches explicitly
when changing template by adding, say, version `v1` to cache key.
now cache key contains MD5 sum of template tree (template file itself and
all of its dependencies) - whenever template changes cache expires.

<https://www.nateberkopec.com/2015/07/15/the-complete-guide-to-rails-caching.html>

**russian doll caching** is simply using key-based cache expiration.

using `touch` option on `belongs_to` associations allows to update
associated object whenever current object is touched or saved.

# turbolinks

<http://guides.rubyonrails.org/working_with_javascript_in_rails.html#turbolinks>
<http://railscasts.com/episodes/390-turbolinks?view=asciicast>
<https://habrahabr.ru/post/167161/>

turbolinks is used to speed up page rendering.

## installation

- install as gem
- require in application js/coffeescript manifest file (_application.coffee_)

turbolinks can be disabled for certain links by adding `data-to-turbolink`
data attribute to the link.

## how it works

- intercepts all clicks on `a` links to the same domain
- changes browser's URL
- requests new page with XHR (Ajax) request
- renders response:
  - replaces current `body` element entirely
  - merges the contents of `head` element
- `window`, `document` objects and `html` element persist between visits

CAUTION: you'll have a long-running, persistent session with maintained state -
pay additional care not leak memory or otherwise bloat that long-running state.

## turbolinks classic

<https://github.com/turbolinks/turbolinks-classic/blob/master/README.md>

### `ready` event

usually `ready` event is used to register event handlers/listeners
on page elements inside a DOM tree.

`ready` event (jQuery specific event - it uses `DOMContentLoaded` event or
emulates it if the former is not available in the browser) occurs when HTML
document is finished loading (or reloading) but content such as images
might not be loaded yet.

but with turbolinks page is not loaded again when clicking the link:
DOM tree has changed but event handlers are not bound to new page elements.

we can fix this problem using turbolinks events: `page:load` or `page:change`.

### `page:load` event

it's possible to fix it by using `page:load` event - it's triggered when:

- page is loaded with turbolinks (new `body` is loaded into the DOM)

BUT it's NOT triggered:

- on partial replacement
- when page is restored from client-side cache (clicking *back* in browser)

### `page:change` event

USE `page:change` instead - it's triggered when:

- page is loaded normally (`ready`, `DOMContentLoaded`)
- page is loaded with turbolinks (`page:load`)
- page is restored from client-side cache (clicking *back* in browser)

in general just replace:

```coffeescript
$(document).ready ->
  alert 'page has loaded!'
```

with

```coffeescript
$(document).on 'page:change', ->
  alert 'page has loaded!'
```

## turbolinks 5

<https://github.com/turbolinks/turbolinks>

### visits

2 types of visits:

- applications visit
- restoration visit

#### application visit

visit actions:

- `advance` (default) - new history entry is created
- `replace` - topmost history entry is replaced

turbolinks caches visited pages - when visiting new page turbolinks first
tries to render preview of the page from cache if it's available while
simultaneously loading a fresh copy of the page from server.

#### restoration visit

visit actions:

- `restore` - shouldn't be invoked manually

if possible, turbolinks renders a copy of the page from cache without
making a request - otherwise retrieves a fresh copy of the page from server.

### `turbolinks:load` event

`turbolinks:load` event fires:

- once on the initial page load
- again after every turbolinks visit
- when navigating by history

```coffeescript
$(document).on 'turbolinks:load', ->
  alert 'page has loaded!'
```

that is `turbolinks:load` is a full replacement of `page:change`.

### non-idempotent transformations

this is the same problem as in turbolinks classic:

non-idempotent transformations shouldn't be run on `turbolinks:load`
since they might modify the same data twice (e.g. insert some element)
when page is restored from cache.

ALL functions inside event handler for `turbolinks:load` must be IDEMPOTENT!

### event listeners

still this doesn't concerns event listeners since they are discarded by
turbolinks when saving a copy of the page to cache (before rendering a new page)
and thusly may be safely bound again when page is restored from cache.

it's recommended to avoid using `turbolinks:load` event to add
event listeners directly to page elements - use **event delegation**
instead to register event listeners once on `document` or `window`.

> **event delegation** refers to the process of using event propagation (bubbling)
> to handle events at a higher level in the DOM than the element on which
> the event originated. it allows us to attach a single event listener for
> elements that exist now or in the future:

```coffeescript
$('#list').on 'click', 'a', (event) ->
  event.preventDefault()
  ...
```
