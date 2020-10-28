---
layout: post
title: "[cpp] 매크로를 사용해 문자열 조합하기"
description: >
  
tags: [cpp, marco, preprocessor]
author: hdmun
comments: true
---



~~~cpp
#ifdef _WIN64
#define __PLATFORM x64
#else
#define __PLATFORM Win32
#endif

#ifdef _DEBUG
#define __BUILD Debug
#else
#define __BUILD Release
#endif

#define __STRINGIFY_(filename) #filename
#define __STRINGIFY(_macro) __STRINGIFY_(_macro)
#define __CONCAT_(x, y) x ## y
#define __CONCAT(x, y) __CONCAT_(x, y)
#define __CONCAT_SEP(x, y) __CONCAT(x, __CONCAT(_, y))
#define __BUILD_PLATFORM __CONCAT_SEP(__BUILD, __PLATFORM)

#include __STRINGIFY( __CONCAT_SEP( HeaderFileName, __CONCAT( __BUILD_PLATFORM, .h ) ) )
~~~
