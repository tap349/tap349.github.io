---
layout: post
title: JavaScript - Flow
date: 2018-01-02 02:28:57 +0300
access: public
comments: true
categories: [js, eslint, flow]
---

<!-- @format -->

<!-- more -->

* TOC
{:toc}
<hr>

see also [JavaScript - ESLint]({% post_url 2017-06-10-javascript-eslint %}).

1. <https://flow.org/en/docs/getting-started/>
2. <https://github.com/facebook/flow>
3. <https://github.com/ryyppy/flow-guide>

## installation

### install Flow locally

```sh
$ yarn add flow-bin --dev
```

### install `babel-preset-flow`

```sh
$ yarn add babel-preset-flow --dev
```

```javascript
// .babelrc

{
  "presets": ["flow"]
}
```

> Now, Babel will always strip away Flow source and your JS runtime can
> interpret the code. This is especially important for feeding into ESlint.

### install `eslint-plugin-flowtype`

```sh
$ yarn add eslint-plugin-flowtype --dev
```

```yaml
# .eslintrc.yml

extends:
  - 'plugin:flowtype/recommended'
plugins:
  - flowtype
```

<https://github.com/ryyppy/flow-guide#eslint-configuration>:

> there will be some warnings about unused variables whenever you write type
> declarations. The eslint plugin eslint-plugin-flowtype will mute those
> warnings.

I don't see any warnings mentioned above but this plugin is still useful: say,
it adds ESLint rule to check for presence of Flow declaration (`// @flow`) if
type annotation is added to the file - ESLint will complain if declaration is
missing:

```
Type annotations require valid Flow declaration.
(flowtype/no-types-missing-file-annotation)
```

this rule `flowtype/no-types-missing-file-annotation` must be a part of
recommended configuration (`plugin:flowtype/recommended`).

`eslint-plugin-flowtype` plugin is ESLint plugin - it's not meant to check for
Flow type errors or to show them! checking for type errors is performed by
`flow` binary (use either `flow status` in terminal or `flow` syntastic JS
checker in Vim).

## configuration

### generate Flow config

```sh
$ yarn run flow init
```

this will create empty _.flowconfig_ in the project root.

## usage

### flag files to be checked with `// @flow`

according to Flow documentation `@flow` annotation can be placed anywhere in JS
file - it just instructs Flow to check current file.

but in my case ONLY the files with `@flow` annotation at the very beginning of
the file are checked by Flow.

<https://github.com/facebook/flow/issues/3316>:

it's possible to use `@flow weak` for weak mode - say, it allows not to specify
types for all parameters.

### ignore Flow errors on a one-off basis

```javascript
// $FlowFixMe
'foo' + {};
```

### add Flow script

```javascript
// package.json

"scripts": {
  "flow": "flow"
},
```

now Flow can be run using any of these commands:

```sh
$ yarn run flow
```

**_UPDATE_**

it's not required to add `flow` script - everything works as is.

### run Flow

- `yarn run flow`

  same as `flow status` - starts Flow server (if not started yet), does a full
  check, prints the results and monitors changes to your code in the background
  checking those changes incrementally for errors - current errors can be shown
  by typing `flow` again.

- `yarn run flow check`

  does a full check in the foreground without starting Flow server.

## Vim integration

### [NOT USED] syntastic

NOTE: Flow must be installed globally for syntastic `flow` checker to be
available (like for ESLint).

```sh
$ yarn global add flow-bin
```

```vim
" ~/.vim/vimrc

let g:syntastic_aggregate_errors = 1

let g:syntastic_javascript_flow_exe = '$(npm bin)/flow focus-check'
let g:syntastic_javascript_checkers = ['flow']
```

<https://medium.com/@renatoagds/flow-vim-the-long-journey-497e020114e5>:

another way to integrate Flow with syntastic is to feed file content into
`flow check-contents` - but now that `flow focus-check` command is added this
method seems to be obsolete.

### [NOT USED] vim-flow

syntastic already performs linting of JS files so linting provided by this
plugin (automatic or manual checks - via `FlowMake` command) seems to be
excessive.

still `FlowJumpToDef` and `FlowType` commands might be useful - just make sure
to disable automatic checks on save.

### ale

unlike syntastic, ALE doesn't require Flow to be installed globally - it finds
Flow script (`$(npm bin)/flow`) somehow.

```vim
" ~/.vim/vimrc

let g:ale_linters = {
      \   'javascript': ['flow']
      \ }
```

## style guide

1. <https://github.com/ryyppy/flow-guide#flow-style-guide>

## notes

### function and property variance

1. <https://flow.org/blog/2016/10/04/Property-Variance/>

<https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)>:

> One function type is a subtype of another when it is safe to use a function of
> one type in a context that expects a function of a different type. It is safe
> to substitute a function f for a function g (substitute function g with
> function f) if f accepts a more general type of arguments and returns a more
> specific type than g.
>
> In other words, the type constructor is contravariant in the input type and
> covariant in the output type.

<https://flow.org/en/docs/lang/variance>:

> Flow has contravariant inputs (accepts less specific types to be passed in),
> and covariant outputs (allows more specific types to be returned).

<https://flow.org/en/docs/lang/depth-subtyping/>:

> By default, object properties are invariant, which allow both reads and
> writes, but are more restrictive in the values they accept (when they are
> declared - my note).

- invariant properties accept specified type only

  invariant properties can be read from and written to:

  ```javascript
  function f(o: {p: string}): number {
    // we can read from o.p
    const len = o.p.length;
    // we can write to o.p
    o.p = 'bar';

    return len;
  }

  var o1: {p: string} = {p: 'foo'};
  f(o1);
  ```

- covariant properties accept specified type or its subtype

  covariant properties are read-only (appear in output positions only):

  ```javascript
  function f(o: {+p: ?string}): number {
    // we cannot write to o.p:
    // o.p might have type that is a subtype of ?string (say, string)
    // but after writing null to o.p, o1.p would no longer have type string
    //o.p = null;
    return o.p ? o.p.length : 0;
  }

  var o1: {p: string} = {p: 'foo'};
  f(o1);
  ```

- contravariant properties accept specified type or its supertype

  contravariant properties are write-only (appear in input positions only):

  ```javascript
  function f(o: {-p: string}): number {
    // we cannot read from o.p:
    // o.p might have type that is a supertype of string (say, ?string)
    // but we don't expect and process other types but string in this
    // function (say, if o.p is null, then o.p.length would raise error)
    //const len = o.p.length;
    o.p = 'default';
  }

  var o1: {p: ?string} = {p: null};
  f(o1);
  ```

### [Type Annotations](https://flow.org/en/docs/types)

> <https://flow.org/en/docs/types/classes/>:
>
> JavaScript classes in Flow operate both as a value and a type. You write
> classes the same way you would without Flow, but then you can use the name of
> the class as a type:
>
> class MyClass { // ... }
>
> let myInstance: MyClass = new MyClass();
>
> This is because classes in Flow are nominally typed.

> <https://flow.org/en/docs/types/interfaces/>:
>
> use `interface` to add structural typing for classes.

> <https://flow.org/en/docs/types/generics/>:
>
> Classes (when being used as a type), type aliases, and interfaces with
> generics are parameterized (all require that you pass type arguments).
> Functions and function types do not have parameterized generics.

say, `React.Component<Props, State>` is a parameterized generic class type (it's
necessary to pass `Props` and `State` types when using this class type).

<https://flow.org/en/docs/types/unions/>:

> Union types requires one in, but all out.

<https://flow.org/en/docs/types/intersections/>:

> Intersection types require all in, but one out.
>
> When you create an intersection of object types, you merge all of their
> properties together.

### [Type System](https://flow.org/en/docs/lang)

> <https://flow.org/en/docs/lang/subtypes/#toc-subtypes-of-functions>:
>
> The function subtyping rule is this: a function type B is a subtype of a
> function type A if and only if B’s inputs are a superset of A’s, and B’s
> outputs are a subset of A’s.

> <https://flow.org/en/docs/lang/nominal-structural>:
>
> Flow uses structural typing for objects and functions, but nominal typing for
> classes.
>
> If you wanted to use a class structurally you could do that by mixing them
> with objects as interfaces.

> <https://flow.org/en/docs/lang/width-subtyping/>:
>
> This is a kind of subtyping commonly referred to as “width subtyping” because
> a type that is “wider” (i.e., has more properties) is a subtype of a narrower
> type.
>
> Exact object types disable width subtyping, and do not allow additional
> properties to exist.

## [React](https://flow.org/en/docs/react/)

- annotate component with `@flow`

  ```javascript
  // @flow
  import * as React from 'react';
  ```

- add `Props` type

  ```javascript
  type Props = {
    isChecked: boolean,
    isDisabled?: boolean,
  };
  ```

  and pass it into `React.Component` as a type argument
  (`React.Component<Props, State>` is a parameterized generic type that takes 2
  arguments):

  ```javascript
  export default class Checkbox extends React.Component<Props> {
    // ...
  }
  ```

  if component extends another custom component (not `React.Component`), specify
  props type inside class body:

  ```javascript
  export default class Checkbox extends BaseCheckbox {
    props: Props;
    // ....
  }
  ```

- add default props as usual

  ```javascript
  export default class Checkbox extends React.Component<Props> {
    static defaultProps = {
      isDisabled: false,
    };
    // ...
  }
  ```

  NOTE: default prop value is not type checked for some reason - you can set it
  to any value. to be precise, its type is infered from `defaultProps` but Flow
  doesn't check that it's the same type as in `Props` type.

* add maybe type for component ref

  1. <https://flow.org/en/docs/react/refs>

  ```jsx
  _form: ?Form;

  render () {
    return <Form ref={ref => this._form = ref} />
  }
  ```

* install `babel-plugin-react-flow-props-to-prop-types` plugin for runtime
  checks (Flow performs static checks only)

  IMPORTANT: don't install this plugin - it causes strange errors (see
  `troubleshooting` section)!

  ```sh
  $ yarn add prop-types prop-types-extra
  $ yarn add babel-plugin-react-flow-props-to-prop-types --dev
  ```

  _.babelrc_:

  ```
  {
    "plugins": [
      ["react-flow-props-to-prop-types"]
    ]
  }
  ```

* don't use decorators - Flow doesn't support them as of now

  1. <https://github.com/facebook/flow/issues/3405>
  2. <https://github.com/agentcooper/react-redux-flow-example/blob/master/src/Counter.js>

  ```javascript
  @connect(
    mapStateToProps,
    mapDispatchToProps,
  )
  export default class MyComponent extends React.Component<Props> {}
  ```

  =>

  ```javascript
  import {connect, type Connector} from 'react-redux';

  type OwnProps = {
    foo: Object,
  };
  type StateProps = {
    bar: ?Object,
  };
  type DispatchProps = {
    getBaz: () => Promise<Object>,
  };
  type Props = OwnProps & StateProps & DispatchProps;

  class MyComponent extends React.Component<Props> {}

  const connector: Connector<OwnProps, Props> = connect(
    mapStateToProps,
    mapDispatchToProps,
  );
  export default connector(MyComponent);
  ```

## types

a very useful example of using Flow types (with their corresponding prop types)
from _REAMDE.md_ of `babel-plugin-react-flow-props-to-prop-types` plugin:

```javascript
type FooProps = {
  an_optional_string?: string,
  a_number: number,
  a_boolean: boolean,
  a_generic_object: Object,
  array_of_strings: Array<string>,
  instance_of_Bar: Bar,
  anything: any,
  mixed: mixed,
  one_of: 'QUACK' | 'BARK' | 5,
  one_of_type: number | string,
  nested_object_level_1: {
    string_property_1: string,
    nested_object_level_2: {
      nested_object_level_3: {
        string_property_3: string,
      },
      string_property_2: string,
    },
  },
  should_error_if_provided: void,
  intersection: {foo: string} & {bar: number} & Qux,
  some_external_type: SomeExternalType,
  some_external_type_intersection: {foo: string} & SomeExternalType,
};
```

## libdefs

1. <https://flow.org/en/docs/libdefs/>

> Using the /flow-typed directory for libdefs is a convention that enables Flow
> to JustWork™ out of the box and encourages consistency across projects that
> use Flow

=> it's not necessary to specify _flow-typed/_ as libdefs directory explicitly
using `[libs]` section of _.flowconfig_.

## debugging

1. <https://github.com/facebook/flow/issues/7277#issuecomment-447848767>

```sh
$ yarn run flow --show-all-branches
```

output gives detailed information about what Flow is complaining about.

## tips

### using with GraphQL

- sync GraphQL fragments (used in queries) to GraphQL types on server
- sync Flow types to GraphQL fragments

  if there are multiple fragments (say, for different use cases) - create
  different Flow types.

- use ad hoc nested Flow types for associations in fragments

  ```flow
  // flow-typed/ExtraPracticeAds.js

  declare type ExtraPracticeAd = {
    +game: {
      +address: ?string,
    },
    +extraPracticeBookings: $ReadOnlyArray<{
      +id: number,
    }>,
  };
  ```

- type nested objects with `$PropertyType` and `$ElementType`

  1. <https://github.com/facebook/flow/issues/4896>
  2. <https://flow.org/en/docs/types/utilities/#toc-propertytype>
  3. <https://flow.org/en/docs/types/utilities/#toc-elementtype>

  > <https://flow.org/en/docs/types/utilities/#toc-elementtype>
  >
  > one of the things that also makes $ElementType<T, K> more powerful than
  > $PropertyType<T, k> is that you can use it with generics.

  => `$ElementType` can be always used in place of `$PropertyType`.

  ```flow
  type OwnProps = {
    game: $PropertyType<ExtraPracticeAd, 'game'>,
    extraPracticeBooking: $ElementType<
      $PropertyType<ExtraPracticeAd, 'extraPracticeBookings'>,
      number,
    >,
  };
  ```

## troubleshooting

### Flow typechecks the whole node_modules/ directory

1. <https://github.com/facebook/flow/issues/1548>

the best solution so far is to install library definitions from `flow-typed`.

### Required module not found

if the whole _node_modules/_ directory is ignored in _.flowconfig_, Flow cannot
find some modules like `react-native` or `react-redux`.

- use more granular ignores in _.flowconfig_

  1. <https://github.com/facebook/flow/issues/1548#issuecomment-198477146>

  that is don't ignore the whole _node_modules/_ directory but do it on per
  package basis so that Flow can find required modules.

- replace not found module with stub module

  1. <https://github.com/facebook/flow/issues/3875#issuecomment-306219044>
  2. <https://github.com/facebook/flow/issues/2092#issuecomment-286399732>

  create _ModuleStub.js_ in the project root:

  ```javascript
  export default {};
  ```

  ```conf
  # .flowconfig

  [options]
  ; modules
  module.name_mapper='^moment$' -> '<PROJECT_ROOT>/ModuleStub'
  module.name_mapper='^react-native$' -> '<PROJECT_ROOT>/ModuleStub'
  module.name_mapper='^react-native-.+$' -> '<PROJECT_ROOT>/ModuleStub'
  module.name_mapper='^react-redux$' -> '<PROJECT_ROOT>/ModuleStub'
  ```

  also it might be necessary to stub all images since their names are replaced
  with `RelativeImageStub` in defalut _.flowconfig_ but this module is not
  available if the whole _node_modules/_ directory is ignored.

- provide external library definitions from `flow-typed` repository

  1. <https://github.com/facebook/flow/issues/1548#issuecomment-318085946>

  ```conf
  # .flowconfig

  [libs]
  <PROJECT_ROOT>/flow-typed
  ```

- create external library definitions by yourself

  1. <https://github.com/facebook/flow/issues/1548#issuecomment-198477146>

  this is like manually creating `flow-typed` repository.

  ```conf
  # .flowconfig

  [libs]
  flow/
  ```

  ```javascript
  // flow/lodash.js

  declare module lodash {
    declare var exports: any;
  }
  ```

### Experimental decorator usage

1. <https://github.com/facebook/flow/issues/606#issuecomment-213667957>

```conf
# .flowconfig

[options]
esproposal.decorators=ignore
```

### Property not found

1. <https://github.com/facebook/flow/issues/1606#issuecomment-267775546>

this is usually a problem when adding new properties to a
[sealed object](https://flow.org/en/docs/types/objects/#toc-sealed-objects).

to fix it cast object to `Object` (it makes this object unsealed?):

```javascript
(object: Object).newProp = 'foo';
```

for class fields in particular this error can be fixed in 2 ways:

- cast class instance (`this`) to its type (see above)

  ```javascript
  export default class NewPage extends Component {
    render() {
      return <Form ref={ref => ((this: NewPage)._form = ref)} />;
      // this works too:
      return <Form ref={ref => ((this: Object)._form = ref)} />;
    }
  }
  ```

- annotate class field within the body of the class

  1. <https://flow.org/en/docs/types/classes/#toc-class-fields-properties>

  ```javascript
  export default class NewPage extends Component {
    _form: Form;

    render() {
      return <Form ref={ref => (this._form = ref)} />;
    }
  }
  ```

### Unexpected super class type: CallExpression

packager log and emulator window:

```
SyntaxError: <APP_DIR>/node_modules/react-native/Libraries/Network/XMLHttpRequest.js:
Unexpected super class type: CallExpression
```

and other cryptic errors that seem to be Flow related.

**solution**

remove `react-flow-props-to-prop-types` package.

```diff
  // .babelrc

  {
    "plugins": [
-     "react-flow-props-to-prop-types"
    ]
  }
```

```sh
$ yarn remove babel-plugin-react-flow-props-to-prop-types --dev
```
