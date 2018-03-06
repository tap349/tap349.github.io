---
layout: post
title: React Native - Updating component
date: 2017-11-25 23:56:19 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

trigger update => compare state and props => render

trigger update
--------------

component's update is triggered when:

- new props are received

  either own props (say, when parent component passes new props to
  its child component as a result of its own state change) or props
  from `mapStateToProps()` and `mapDispatchToProps()` which are
  merged into the component's props (when component is connected).

  NOTE: when component is created inside navigator scene, it longer
        receives new props which are usually passed as route params
        unless that scene is rendered again explicitly (say, when
        pushing its route into the navigator stack). in other words,
        chain of updates doesn't survive navigator transitions.

- new state is received

  `setState()` is called inside component.

- Redux store is updated

  <https://github.com/reactjs/react-redux/blob/master/docs/api.md#arguments>:

  > connected component is subscribed to Redux store updates:
  > any time the store is updated, `mapStateToProps()` will be called.

- `forceUpdate()` is called

compare state and props
-----------------------

1. <https://reactjs.org/docs/react-component.html#shouldcomponentupdate>

comparison is made inside `shouldComponentUpdate()` which is

- invoked before rendering when new props or state are received
- `true` by default (always re-render component)
- not called for initial render or when `forceUpdate()` is used
- implemented by `React.PureComponent` with shallow prop and state comparison

  <https://reactjs.org/docs/react-api.html#reactpurecomponent>:

  > React.PureComponent’s shouldComponentUpdate() only shallowly
  > compares the objects. If these contain complex data structures,
  > it may produce false-negatives for deeper differences.

  <https://hackernoon.com/react-purecomponent-considered-harmful-8155b5c1d4bc>:

  when using `React.PureComponent`, performance might even degrade a little
  bit if some object that is passed as property value is created in place =>
  `shouldComponentUpdate()` will always return `true` but still it has to
  iterate over all keys of both props and nextProps on each update.

- implemented by `connect()` with shallow prop and state comparison

  1. <https://github.com/reactjs/react-redux/blob/3.x/src/components/createConnect.js#L91>
  2. <https://stackoverflow.com/questions/38189783>

  <https://github.com/reactjs/react-redux/blob/master/docs/troubleshooting.md#my-views-arent-updating-when-something-changes-outside-of-redux>:

  > connect() implements shouldComponentUpdate by default, assuming that your
  > component will produce the same results given the same props and state.

  <http://redux.js.org/docs/faq/ReactRedux.html#react-not-rerendering>:

  > React Redux tries to improve performance by doing shallow equality reference
  > checks on incoming props in shouldComponentUpdate, and if all references are
  > the same, returns false to skip actually updating your original component.
  >
  > It's important to remember that whenever you update a nested value,
  > you must also return new copies of anything above it in your state tree.
  > If you have state.a.b.c.d, and you want to make an update to d,
  > you would also need to return new copies of c, b, a, and state.

  <https://github.com/reactjs/redux/issues/585#issuecomment-244184656>:

  > Typically problems arise when you write reducers in such a way that
  > the object identity of a piece of state doesn't change, while some
  > nested attribute within it does change. This is mutating state rather
  > than returning new state immutably, which causes react-redux to think
  > nothing changed, when in fact, something did change.

  <https://github.com/reactjs/redux/issues/585#issuecomment-132865158>:

  > One of the most common reasons that your components might not be
  > re-rendering is that you're modifying the existing state in your
  > reducer instead of returning a new copy of state with the necessary
  > changes.

NOTE: in all cases where state or props are compared, shallow comparison is
      used - only prop references are compared! if you pass mutated property
      (say, some nested value is updated), it'll be considered unchanged =>
      always return new objects from reducers!

### connected component vs. PureComponent

connected component behaves just like `PureComponent`:

- they both implement shallow prop and state comparison inside
  `shouldComponentUpdate()`
- they are both re-rendered when `forceUpdate()` is called
  (since `shouldComponentUpdate()` is bypassed then)

the only difference is that connected component is subscribed
to Redux store updates while `PureComponent` is not (so update
of the latter is triggered only when its props or state change).

render
------

1. <https://medium.com/@gethylgeorge/how-virtual-dom-and-diffing-works-in-react-6fc805f9f84e>

`render()` is called every time `shouldComponentUpdate()` returns true.

rendering the component is:

1. updating virtual DOM
2. running diffing algorithm
3. updating actual DOM

so even if you call `forceUpdate()`, say, on every Redux store update, it
won't force update actual DOM every time unless markup has really changed
(and only the parts that correspond to changed markup will get updated).
