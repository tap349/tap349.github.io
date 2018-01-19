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

React Developer Tools
---------------------

1. <https://facebook.github.io/react-native/docs/debugging.html#react-developer-tools>

it's a standalone version of an official Remote Debugger -
Chrome Developer Tools (CDT).

`react-devtools` connects to simulator automatically
(additional configuration for Android is required).

Remote Redux DevTools
---------------------

1. <https://github.com/zalmoxisus/remote-redux-devtools>
2. <https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd>

Chrome: `Redux DevTools` extension icon menu -> `Open Remote DevTools`

[RECOMMENDED] React Native Debugger (RND)
-----------------------------------------

1. <https://github.com/jhen0409/react-native-debugger>
2. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/getting-started.md>

RND contains:

- React Developer Tools (CDT - RND is based on CDT)
- React DevTools
- Redux DevTools

so RND is an all-in-one solution - there's no need to install
other tools separately.

### use RND as Remote Debugger

1. <https://facebook.github.io/react-native/docs/debugging.html#debugging-using-a-custom-javascript-debugger>

Remote Debugger is opened automatically when application is launched
and `Debug JS Remotely` option is enabled in Developer Menu.

_~/.zshenv_:

```zsh
export REACT_DEBUGGER="open -g 'rndebugger://set-debugger-loc?host=localhost&port=8081' --args"
```

adding `--args` option in the end prevents from opening current application
directory in Finder after launching React Native Debugger.
