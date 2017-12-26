---
layout: post
title: React Native - Generating release
date: 2017-12-26 17:52:35 +0300
access: public
comments: true
categories: [react-native]
---

<!-- more -->

* TOC
{:toc}
<hr>

iOS
---

- open _iceperkapp.xcworkspace_ in Xcode
- select `Generic iOS Device`
- increment build number: `General` -> `Identity` -> `Build`
  (it should be incremented synchronously for Android too)
- create archive: `Product` (top menu) -> `Archive`

troubleshooting
---------------

### Invalid bitcode version (Producer: '802.0.41.0_0' Reader: '800.0.42.1_0')

build failed in Xcode:

```
error: Invalid bitcode version (Producer: '802.0.42.0_0' Reader: '800.0.42.1_0')
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

**solution**

1. <https://stackoverflow.com/questions/43480556/xcode-8-2-1-error-invalid-bitcode-version-producer-802-0-41-0-0-reader>

either:

- *[RECOMMENDED]* update Xcode to 8.3
- disable bitcode in `Build Settings` -> `Build Options` -> `Enable Bitcode`
