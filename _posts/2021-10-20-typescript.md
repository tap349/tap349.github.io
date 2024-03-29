---
layout: post
title: Typescript
date: 2020-10-20 10:26:08 +0300
access: public
comments: true
categories: []
---

<!-- more -->

* TOC
{:toc}
<hr>

### Compatibility with ESLint

1. <https://stackoverflow.com/a/63239805/3632318>

```sh
$ yarn add --dev @typescript-eslint/parser
```

```diff
  module.exports = {
-   parser: 'babel-eslint',
+   parser: '@typescript-eslint/parser',
  }
```

### Material-UI

1. <https://mui.com/customization/theming/#custom-variables>

```typescript
// src/theme.tsx

declare module '@material-ui/core/styles/createMuiTheme' {
  interface Theme {
    custom_palette: {
      backdrop: {
        main: string;
      };
    };
    custom_typography: {
      link: {};
    };
  }

  interface ThemeOptions {
    custom_palette?: {
      backdrop?: {
        main?: string;
      };
    };
    custom_typography?: {
      link?: Object;
    };
  }
}

export default createMuiTheme({
  custom_palette: {
    backdrop: {
      main: 'rgba(10, 10, 10, 0.9)',
    },
  },
  custom_typography: {
    link: {
      color: '#A6A6A6',
      fontSize: 14,
      // ...
    },
  },
});
```
