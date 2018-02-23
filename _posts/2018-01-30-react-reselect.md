---
layout: post
title: React - Reselect
date: 2018-01-30 03:06:46 +0300
access: private
comments: true
categories: [react, reselect]
---

<!-- more -->

* TOC
{:toc}
<hr>

notes
-----

<https://medium.com/@parkerdan/react-reselect-and-redux-b34017f8194c>:

> In the real world, you will most likely need the same certain part of your
> state object in multiple components. You will also want to pass props to
> your selector. To do this, you need to create a selector function that can
> be used on multiple instances of the same component at the same time.

### mapStateFromProps vs. reselect + mapStateFromProps

if it's necessary to get data from Redux store (or computed data based on
data from Redux store), wrap component in connected container and provide
this data via `mapStateFromProps`.

if it's necessary to get the same computed data from Redux store but
computing this data on each component update is costly, fetch it via
selector (where it's cached) and provide further via `mapStateFromProps`
as usual.

### input selectors are always computed

1. <https://github.com/toomuchdesign/re-reselect/blob/master/examples/1-join-selectors.md>

try to use selectors created with `createSelector` as input selectors
so that they are not recomputed on each call.

consider this example:

```javascript
export const getTeamGames = createCachedSelector(
  (state, teamId) => state.games.all.filter(v => v.team_id === teamId),
  (games) => games,
)((_state, teamId) => teamId);
```

even though cached selector is returned for each `teamId` but then
input selector is always computed and the most expensive operation
here - filtering - is performed on each call.

this example can be refactored into:

```javascript
export const getTeamGames = createCachedSelector(
  [
    getGames,
    (state, teamId) => teamId,
  ],
  (games, teamId) => games.filter(v => v.team_id === teamId),
)((_state, teamId) => teamId);
```

here filtering happens inside result function (not input selector)
and therefore properly cached.

troubleshooting
---------------

### selector is always recomputing when input state stays the same

I had this code:

```javascript
const getProfilesByUserId = (state) => (
  state.profiles.all.reduce((acc, v) => {
    acc[v.user_id] = v;
    return acc;
  }, {})
);

export const getGamersWithProfiles = createCachedSelector(
  [getGamers, getProfilesByUserId],
  (gamers, profilesByUserId, _gameId) => (
    gamers.map(v => ({...v, profile: profilesByUserId[v.id]}))
  ),
)((state, gameId) => gameId);
```

the problem is that `getProfilesByUserId` input selector always returns new
object causing `getGamersWithProfiles` selector to recompute on each update.

**solution**

input selector must return an existing object (a slice of Redux store state)
that changes only when state changes =\> don't return a new manually created
object from input selector.

refactored code:

```javascript
const getProfiles = (state) => state.profiles.all;

export const getGamersWithProfiles = createCachedSelector(
  [getGamers, getProfiles],
  (gamers, profiles, _gameId) => {
    const profilesByUserId = getProfilesByUserId(profiles);
    return gamers.map(v => ({...v, profile: profilesByUserId[v.id]}));
  },
)((state, gameId) => gameId);

//------------------------------------------------------------------------------
// helpers
//------------------------------------------------------------------------------

const getProfilesByUserId = (profiles) => (
  profiles.reduce((acc, v) => {
    acc[v.user_id] = v;
    return acc;
  }, {})
);
```
