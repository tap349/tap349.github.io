---
layout: post
title: React - Redux
date: 2017-06-01 16:52:49 +0300
access: public
categories: [react, react-native, redux]
---

<!-- more -->

<http://redux.js.org/docs/basics/>

> The whole state of your app is stored in an object tree inside a single store.

> The only way to change the state tree is to emit an action, an object describing what happened.

> To specify how the actions transform the state tree, you write pure reducers.

## actions

> Actions are payloads of information that send data from your application
> to your store. They are the only source of information for the store.
> You send them to the store using store.dispatch().

```javascript
{
  type: TOGGLE_TODO,
  index: 5
}
```

> Action creators are functions that create actions.

```javascript
function addTodo(text) {
  return {
    type: TOGGLE_TODO,
    text
  }
}
```

## reducers

> The reducer is a pure function that takes the previous state and an action,
> and returns the next state.

```javascript
(previousState, action) => newState
```

> Things you should never do inside a reducer:

- mutate its arguments
- produce side effects like API calls and routing transitions
- call non-pure functions (say, `Date.now()` or `Math.random()`)

#### reducer composition

<http://redux.js.org/docs/basics/Reducers.html#splitting-reducers>

reducer composition (fundamental pattern of Redux) -
delegating a slice of state to manage to child reducers.

NOTE: it's possible to nest child reducers! say, child reducer to set `user`
      state field might call another child reducer to set user's name.

every reducer on any level receives part of state it manages on this level
and current action and should return the same part of state with merged changes.

without reducer composition:

- new values for state fields are merged into global state in `case` clauses
  for each action type (different action types change different state fields)
- initial state is a global constant and is used as `state` parameter default
  value in main/root (and the only) reducer

with reducer composition:

- each state field is managed independently by dedicated child reducer
  (which receives only part of state it manages and current action)
- initial state of state fields is managed by corresponding child reducers

classic style (without reducer composition):

```javascript
const initialState = {
  count: 0
}

export default function badges(state = initialState, action = {}) {
  switch(action.type) {
    case BADGES_SET_NOTI_COUNT:
      return {
        ...state,
        notiCount: action.count
      }
    default:
      return state
  }
}
```

with reducer composition (but without using `combineReducers`):

```javascript
function notiCount(state = 0, action) {
  switch(action.type) {
    case BADGES_SET_NOTI_COUNT:
      return action.count
    default:
      return state
  }
}

export default function badges(state, action) {
  return {
    notiCount: notiCount(state.notiCount, action)
  }
}
```

with reducer composition (using `combineReducers`):

```javascript
import { combineReducers } from 'redux'

function notiCount(state = 0, action) {
  switch(action.type) {
    case BADGES_SET_NOTI_COUNT:
      return action.count
    default:
      return state
  }
}

const badges = combineReducers(
  {
    notiCount
  }
)

export default badges
```

## store

- create (say, in _Store.js_):

  ```javascript
  import { createStore } from 'redux'
  import badges from './reducers'

  export default createStore(badges)
  ```

- get current state:

  ```javascript
  import store from './Store'

  let state = store.getState()
  ```

- change state by dispatching actions:

  ```javascript
  import store from './Store'
  import * as badgeActions from './actions/badgeActions'

  store.dispatch(badgeActions.setNotiCount(3))
  ```

- listen to state updates:

  ```javascript
  import store from './Store'

  store.subscribe(() => this.forceUpdate())
  ```

  every time state changes listener is called
  (which re-renders root component in this example).

## tips

### share state between reducers

<https://stackoverflow.com/questions/35375810>

say, you want to calculate total count and save it as a state field in store.

> One of the golden rules of Redux is that you should try to avoid putting
> data into state if it can be calculated from another state, as it increases
> likelihood of getting data that is out-of-sync.

### don't dispatch in reducer

<https://stackoverflow.com/questions/36730793/dispatch-action-in-reducer>

> Dispatching an action within a reducer is an anti-pattern.
> Reducer should be without side effects simply digesting the action payload
> and returning a new state object. Adding listeners and dispatching actions
>>  within the reducer can lead to chained actions and other side effects.
