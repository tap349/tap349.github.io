---
layout: post
title: React - Redux
date: 2017-06-01 16:52:49 +0300
access: public
comments: true
categories: [react, react-native, redux]
---

<!-- more -->

* TOC
{:toc}
<hr>

<https://github.com/reactjs/redux>:

> - The whole state of your app is stored in an object tree inside a single store.
> - The only way to change the state tree is to emit an action,
>   an object describing what happened.
> - To specify how the actions transform the state tree, you write pure reducers.

store
-----

- create (say, in _Store.js_):

  ```javascript
  import {createStore} from 'redux';
  import badges from './reducers';

  export default createStore(badges);
  ```

- get current state:

  ```javascript
  const state = store.getState();
  ```

- change state by dispatching actions:

  ```javascript
  store.dispatch(badgeActions.setCount(3));
  ```

- subscribe to state updates:

  ```javascript
  store.subscribe(() => this.forceUpdate());
  ```

  every time state changes listener function is called
  (which re-renders root component with all its child components here).

actions
-------

> Actions are payloads of information that send data from your application
> to your store. They are the only source of information for the store.
> You send them to the store using store.dispatch().

```javascript
{
  type: SET_COUNT,
  count: 5,
}
```

> Action creators are functions that create actions.

```javascript
function setCount (count) {
  return {
    type: SET_COUNT,
    count,
  };
}
```

reducers
--------

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
  count: 0,
};

export default function badges (state = initialState, action = {}) {
  switch (action.type) {
  case SET_COUNT:
    return {
      ...state,
      count: action.count,
    };
  default:
    return state;
  }
}
```

with reducer composition (but without using `combineReducers`):

```javascript
function count (state = 0, action) {
  switch (action.type) {
  case SET_COUNT:
    return action.count;
  default:
    return state;
  }
}

export default function badges (state, action) {
  return {count: count(state.count, action)};
}
```

with reducer composition (using `combineReducers`):

```javascript
import {combineReducers} from 'redux';

function count (state = 0, action) {
  switch (action.type) {
  case SET_COUNT:
    return action.count;
  default:
    return state;
  }
}

const badges = combineReducers({count});

export default badges;
```

react-redux
-----------

1. <https://github.com/reactjs/react-redux/blob/master/docs/api.md>
2. <http://redux.js.org/docs/faq/ReactRedux.html>
3. <http://rants.broonix.ca/getting-started-with-react-native-and-redux/>
4. <http://www.sohamkamani.com/blog/2017/03/31/react-redux-connect-explained/>
5. <https://goshakkk.name/redux-antipattern-mapstatetoprops/>
6. <https://stackoverflow.com/a/40068198/3632318>

using react-redux boils down to using just 2 things:

- `Provider` component

  wraps some parent component (not necessarily root component) and
  provides Redux store to all its child components.

- `connect()` function

  1. <https://stackoverflow.com/questions/32646920/whats-the-at-symbol-in-the-redux-connect-decorator>
  2. <https://stackoverflow.com/a/41438191/3632318>

  to connect component (to Redux store) ==
  to wrap component in HOC (container) using `connect()`.

  simplified implementation of `connect()`:
  [connect.js](https://gist.github.com/gaearon/1d19088790e70ac32ea636c025ba424e).

  connects specified child component to Redux store by passing additional
  properties to component - its purpose is not just to pass a state subtree
  to component but to translate state structure into what component needs
  so that Redux details are not leaked into component (component shouldn't
  use Redux store directly!).

  `connect()` takes 2 arguments (well, actually more - see docs):

  - `mapStateToProps` function

    function is given Redux store as an argument - return
    a plain object that will be merged into component's props.

    also component will subscribe to Redux store updates:
    any time store is updated, `mapStateToProps` will be called.

    if `mapStateToProps` is passed the 2nd argument `ownProps` it
    will be called when store is updated OR when `ownProps` differ
    (since new props may affect return value of `mapStateToProps`).

  - `mapDispatchToProps` function

    function is given `dispatch` function as an argument - return
    a plain object that binds action creators using `dispatch`
    (to bind action creator is to create anonymous function that
    calls `dispatch` which is passed some action - usually created
    with some action creator accordingly) - this can also be done
    using `bindActionCreators` helper from `redux` package.

  think of `connect()` as a Redux store facade for component.

  state \<=\> `connect()` \<=\> component

  ```javascript
  import {connect} from 'react-redux';

  const mapStateToProps = (state, ownProps) => {
    const {team_id} = ownProps.bill;
    return {team: StoreHelper.findTeam(state, team_id)};
  };

  class MyComponent extends Component {...}

  export default connect(mapStateToProps)(MyComponent);
  ```

  or else using decorator:

  ```javascript
  import {connect} from 'react-redux';

  const mapStateToProps = (state, ownProps) => {
    const {team_id} = ownProps.bill;
    return {team: StoreHelper.findTeam(state, team_id)};
  };

  @connect(mapStateToProps)
  class MyComponent extends Component {...}
  ```

tips
----

### don't share state between child reducers

1. <https://stackoverflow.com/questions/35375810>

say, you want to calculate total count and save it as a state field in store.

> One of the golden rules of Redux is that you should try to avoid putting
> data into state if it can be calculated from another state, as it increases
> likelihood of getting data that is out-of-sync.

### don't dispatch in reducer

1. <https://stackoverflow.com/questions/36730793/dispatch-action-in-reducer>

> Dispatching an action within a reducer is an anti-pattern.
> Reducer should be without side effects simply digesting the action payload
> and returning a new state object. Adding listeners and dispatching actions
> within the reducer can lead to chained actions and other side effects.

### don't pass store to presentational components

1. <https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0>
2. <https://redux.js.org/docs/basics/UsageWithReact.html#presentational-and-container-components>

only container components are aware of store and provide data from store to
presentational and other container components.

that is why only container components should be connected to Redux store.

### initialize state in reducers with ES6 default arguments

<http://redux.js.org/docs/recipes/reducers/InitializingState.html#single-simple-reducer>:

> When Redux initializes it dispatches a "dummy" action to fill the state.
> This is exactly the case that "activates" the default argument.

```javascript
function isModalVisible (state = false, action) {
  switch (action.type) {
  case SHOW_MODAL:
    return true;
  case HIDE_MODAL:
    return false;
  default:
    return state;
  }
}

export default combineReducers({isModalVisible});
```

or when not using reducer composition:

```javascript
const initialState = {
  isModalVisible: false,
}

export default function memberships (state = initialState, action = {}) {
  switch (action) {
  case SHOW_MODAL:
    return {...state, isModalVisible: true};
  case HIDE_MODAL:
    return {...state, isModalVisible: false};
  default:
    return state;
  }
}
```

### store object keyed by ID instead of array

- <http://redux.js.org/docs/recipes/reducers/NormalizingStateShape.html>
- <https://medium.com/dailyjs/rewriting-javascript-converting-an-array-of-objects-to-an-object-ec579cafbfc7>

{% raw %}
```javascript
// [{id: 2, name: 'foo'}, {id: 2, name: 'bar'}] ->
//  {2: {id: 1, name: 'foo'}, 2: {id: 2, name: 'bar'}}
const arrayToObject = (array) =>
  array.reduce((obj, item) => {
    obj[item.id] = item;
    return obj;
  }, {})
```
{% endraw %}

NOTE: original array sorting is lost in resulting object!

if you need to keep sorting opt for `Map` instead:

```javascript
const arrayToMap = (array) =>
  array.reduce((map, item) => {
    map.set(item.id, item);
    return map;
  }, new Map());
```

also it might be more convenient to store both object keyed by ID
(`byId` store key) and array of all items (`all` store key).

### don't store refreshing flag in Redux store

this causes flickering of `RefreshControl` component during animation -
store it in component state instead.

style guide
-----------

### thunk actions

```javascript
// pass `params` object when updating the whole model
export const requestUpdateGame = (id, params) => (
  (dispatch, getState, api) => {
    // ...
    return api.updateGame(token, id, params)
    // ...
  }
)

// pass specific attribute when updating that attribute
export const requestUpdateGameKind = (id, kind) => (
  (dispatch, getState, api) => {
    // ...
    return api.updateGame(token, id, {kind})
    // ...
  }
)
```

middleware
----------

### [Redux Thunk](https://github.com/gaearon/redux-thunk)

<https://github.com/gaearon/redux-thunk#composition>:

> A thunk is a function that returns a function.

<https://github.com/reactjs/redux/issues/1676#issuecomment-215413478>

> The return value of dispatch() when you dispatch a thunk *is*
> the return value of the inner function. This is why it's useful
> to return a Promise (even though it is not strictly necessary)

that is dispatching a thunk returns whatever thunk itself returns -
not necessarily `Promise` object (even though it's highly recommended).

#### handling rejected promises in thunks

in thunk:

```javascript
export const requestPlayers = (teamId) => (
  (dispatch, getState, api) => {
    dispatch(startLoading());

    const {token} = getState().user.credentials;

    return api.getPlayers(token, teamId)
      .then(data => {
        dispatch(set(data.players));
        return data.players;
      })
      .catch(e => {
        dispatch(finishLoading());
        Log.info(e.message);
        throw e;
      });
  }
);
```

don't forget to re-throw error in `catch` method body to return rejected promise.

in component:

```javascript
// notify user about error
this.props.store
  .dispatch(teamsActions.requestPlayers(this.props.team.id))
  .catch(_e => AlertHelpers.serverError());

// or else silence error
this.props.store
  .dispatch(teamsActions.requestPlayers(this.props.team.id))
  .catch(_e => {});
```

in any case it's required to add `catch` method call in component
in order to avoid warning about unhandled promise rejection.

debugging
---------

see [React Native - Debugging]({% post_url 2017-11-14-react-native-debugging %}).

troubleshooting
---------------

### component is not re-rendered when it's connected

see the section above about updating component when using react-redux.

in my case connected component `GamerCheckedRow` is passed gamer and
callback to calculate if gamer is checked or not. when gamer is clicked,
`selected_user_ids` state property of parent component is updated inside
passed callback - not the gamer himself. but `selected_user_ids` state
property is not passed as a property of `GamerCheckedRow` component so
React thinks that props of `GamerCheckedRow` have not changed and doesn't
re-render it (it would re-render if, say, `forceUpdate()` would be called).

**solution**

there are 2 ways to solve this problem:

- pass `pure: false` option to `connect()`

  <https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md#my-views-arent-updating-when-something-changes-outside-of-redux>:

  > This will remove the assumption that GamerCheckedRow is pure
  > and cause it to update whenever its parent component renders.
  > Note that this will make your application less performant,
  > so only do this if you have no other option.

  ```javascript
  @connect(mapStateToProps, null, null, {pure: false})
  ```

- pass calculated `isChecked` property instead of a callback to calculate it

  because the former changes when new gamers are selected while the latter
  doesn't - IMO this should be a prefered approach to solve the problem.

### functions of connected component are not available from outside

1. <https://github.com/reactjs/react-redux/issues/475>

say, when connected component is obtained via its `ref` property.

**solution**

connected component:

```javascript
@connect(mapStateToProps, null, null, {withRef: true})
export default class MyComponent extends Component {
  // ...
}
```

calling its function from outside:

```javascript
this.myComponent.getWrappedInstance().tryScrollToGame(games[0]);
```
