---
layout: post
title: React Native - Debugging
date: 2017-11-14 16:23:55 +0300
access: public
comments: true
categories: [react-native, redux]
---

<!-- @format -->

<!-- more -->

<!-- prettier-ignore -->
* TOC
{:toc}
<hr>

1. <https://facebook.github.io/react-native/docs/debugging.html>

## React Developer Tools

1. <https://facebook.github.io/react-native/docs/debugging.html#react-developer-tools>

it's a standalone version of an official Remote Debugger - Chrome Developer
Tools (CDT).

`react-devtools` connects to simulator automatically (additional configuration
for Android is required).

## Remote Redux DevTools

1. <https://github.com/zalmoxisus/remote-redux-devtools>
2. <https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd>

| Chrome                                                     |
| ---------------------------------------------------------- |
| `Redux DevTools` (extension menu) → `Open Remote DevTools` |

## _[RECOMMENDED]_ React Native Debugger (RND)

1. <https://github.com/jhen0409/react-native-debugger>
2. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/getting-started.md>

RND contains:

- Developer Tools (CDT - RND is based on CDT)
- React DevTools
- Redux DevTools

so RND is an all-in-one solution - there's no need to install other tools
separately.

### use RND as Remote Debugger

1. <https://facebook.github.io/react-native/docs/debugging.html#debugging-using-a-custom-javascript-debugger>
2. <https://github.com/jhen0409/react-native-debugger/blob/master/docs/getting-started.md#launch-by-cli-or-react-native-packager-macos-only>
3. <https://github.com/artsy/emission/blob/45417ca425f2cba7d2da21902ef8ff1cd093a024/package.json#L28>

Remote Debugger is opened automatically when application is launched and
`Debug JS Remotely` option is enabled in Developer Menu.

_~/.zshenv_:

```zsh
export REACT_DEBUGGER="open -g 'rndebugger://set-debugger-loc?host=localhost&port=8081' --args"
```

adding `--args` option in the end prevents from opening current application
directory in Finder after launching React Native Debugger.

### record network log

| Redux DevTools                 |
| ------------------------------ |
| RMB → `Enable Network Inspect` |

| Developer Tools                                 |
| ----------------------------------------------- |
| `Network` (tab) → `Record network log` (button) |

it's not sufficient to press `Record network log` button only.

### window configuration tips

it's convenient to dock Developer Tools to bottom and hide either React or Redux
DevTools - this way there's enough space to the right of RDN to place emulator
side by side with RDN.

IMO there's no sense in displaying both React and Redux DevTools at the same
time - show/hide React DevTools (Inspector) with `<D-M-j>` and Redux DevTools
with `<D-M-k>` on demand (BTW it's easier to use `Command` and `Option` keys to
the right of `Space` key instead of the ones to the left).

when React DevTools are hidden, emulator's built-in Inspector is used which is
also very handy - turn on React DevTools only when it's strictly necessary to
explore React component hierarchy in more detail (I find built-in Inspector more
convenient for simple cases since results are shown right in emaulator's
window).

## Reactotron

1. <https://github.com/infinitered/reactotron>
