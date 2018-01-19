---
layout: post
title: React Native - Debugging
date: 2017-11-14 16:23:55 +0300
access: public
comments: true
categories: [react-native, redux]
---

<!-- more -->

* TOC
{:toc}
<hr>

1. <https://facebook.github.io/react-native/docs/debugging.html>

Remote Redux DevTools
---------------------

1. <https://github.com/zalmoxisus/remote-redux-devtools>
2. <https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd>

Chrome: `Redux DevTools` extension icon menu -> `Open Remote DevTools`

React Developer Tools
---------------------

1. <https://facebook.github.io/react-native/docs/debugging.html#react-developer-tools>

`react-devtools` connects to simulator automatically
(additional configuration for Android is required).

React Native Debugger
---------------------

1. <https://github.com/jhen0409/react-native-debugger>
2. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/getting-started.md>

NOTE: this is the only debugger I'm currently using.

### use as custom JavaScript debugger

1. <https://facebook.github.io/react-native/docs/debugging.html#debugging-using-a-custom-javascript-debugger>

- Chrome Developer Tools are default JavaScript debugger
- JavaScript debugger is opened automatically when application is
  launched and `Debug JS Remotely` option is enabled in Developer Menu

_~/.zshenv_:

```zsh
export REACT_DEBUGGER="open -g 'rndebugger://set-debugger-loc?host=localhost&port=8081' --args"
```

adding `--args` option in the end prevents from opening current application
directory in Finder after launching React Native Debugger.
