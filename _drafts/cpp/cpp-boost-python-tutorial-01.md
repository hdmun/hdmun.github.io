---
layout: post
title: "boost python으로 C/C++ 코드 바인딩"
description: >
  
tags: [dev, cpp, pytho, boost]
author: hdmun
comments: true
---

# Boost 라이브러리 빌드

## 다운로드

https://www.boost.org/users/history/version_1_71_0.html


## 빌드하기


### project-config.jam
~~~jam

import option ; 
 
using msvc ; 
using python : 3.7 : %USERPROFILE%/AppData/Local/Programs/Python/Python37 : %USERPROFILE%/AppData/Local/Programs/Python/Python37/include : %USERPROFILE%/AppData/Local/Programs/Python/Python37/libs ;

option.set keep-going : false ; 
~~~

~~~cmd
b2 -j4 --toolset=msvc-14.0 --build-type=complete architecture=x86 address-model=64 stage
~~~
