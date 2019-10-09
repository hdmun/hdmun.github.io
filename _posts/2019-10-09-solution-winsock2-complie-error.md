---
layout: post
title: "WinSock2.h 사용시 재정의 빌드 에러 해결법"
description: >
 간단한 해결법이지만 자꾸 까먹어서 기록해놓는다.
tags: [dev, windows]
author: hdmun
comments: true
---

`WinSock2.h` 파일을 include 했을 때 빌드 에러가 발생하는 경우가 생긴다. 검색해서 들어오셨다면 일단 해결법이 우선이니 해결법을 제시한다.

# 해결법

해결 방법은 두 가지다. 원하는 걸로 골라서 쓰면된다.

## include 순서 변경

`Windows.h`와 같이 사용할 경우 `include` 순서를 `WinSock2.h`보다 아래로 두면 된다.

~~~cpp
#include <WinSock2.h>
// ...
#include <Windows.h>
~~~

## WIN32_LEAN_AND_MEAN 매크로 정의

순서는 그대로 두고 `Windows.h` 위에 `WIN32_LEAN_AND_MEAN` 매크로를 정의해주면 된다.

~~~cpp
#define WIN32_LEAN_AND_MEAN

#include <Windows.h>
#include <WinSock2.h>
~~~


# 왜 저렇게 써야하는 걸까?

해결 방법은 알았으나 뭔가 찜찜하다. 왜 헤더 순서를 바꾸거나 `WIN32_LEAN_AND_MEAN` 매크로를 정의해야 하는 걸까?

답은 `Windows.h` 파일 안에 있다. 긴말할 거 없이 `Windows.h` 파일을 열어서 `WIN32_LEAN_AND_MEAN` 매크로를 사용하는 곳을 찾아보자.

~~~cpp
// Windows.h
// ...
#ifndef WIN32_LEAN_AND_MEAN
#include <cderr.h>
#include <dde.h>
#include <ddeml.h>
#include <dlgs.h>
#ifndef _MAC
#include <lzexpand.h>
#include <mmsystem.h>
#include <nb30.h>
#include <rpc.h>
#endif
#include <shellapi.h>
#ifndef _MAC
#include <winperf.h>
#include <winsock.h>
#endif
#ifndef NOCRYPT
#include <wincrypt.h>
#include <winefs.h>
#include <winscard.h>
#endif

#ifndef NOGDI
#ifndef _MAC
#include <winspool.h>
#ifdef INC_OLE1
#include <ole.h>
#else
#include <ole2.h>
#endif /* !INC_OLE1 */
#endif /* !MAC */
#include <commdlg.h>
#endif /* !NOGDI */
#endif /* WIN32_LEAN_AND_MEAN */
// ...
~~~

뭘 많이 `include` 하고 있다. `include` 리스트 중에 `winsock.h` 파일이 있다. 윈도우 소켓 API 1 버전인데 `Windows.h` 파일은 1버전을 기본으로 `include` 하고 있다.

`WIN32_LEAN_AND_MEAN` 매크로없이 `#include <Windows.h>`를 하게되면 1 버전 헤더 파일이 포함되기 때문에 이름이 겹치는 윈도우 소켓 API 함수들이 재정의 빌드 에러가 발생하게 되는 것이다.

그렇다면 `WinSock2.h` 파일을 먼저 `include` 했을 때는 왜 빌드 에러가 발생하지 않는걸까?

그에 대한 답은 `_WINSOCKAPI_` 매크로에 있다. `winsock.h`, `WinSock2.h` 파일을 열어보자. 두 파일 모두 아래와 같이 처리되어있는 부분이 있다.

~~~cpp
// winsock.h
#ifndef _WINSOCKAPI_
#define _WINSOCKAPI_

// ...

#endif  /* _WINSOCKAPI_ */
~~~

~~~cpp
// WinSock2.h
#ifndef _WINSOCK2API_
#define _WINSOCK2API_
#define _WINSOCKAPI_   /* Prevent inclusion of winsock.h in windows.h */

// ...

#endif  /* _WINSOCK2API_ */
~~~

헤더 중복 포함문제를 해결하기 위해 `#pragma once` 대신 매크로를 사용한 걸 볼 수 있다.

`WinSock2.h`을 먼저 포함하게 되면 `winsock.h`에서 사용하고 있는 `_WINSOCKAPI_` 매크로를 정의해버려 코드가 포함되지 않아. 재정의 빌드 에러가 발생하지 않는 것이다.


# 결론

`Windows.h`를 먼저 포함하면서 `winsock.h` 관련된 재정의 빌드 에러만 없애고 싶은 사람은 `WIN32_LEAN_AND_MEAN` 매크로 대신 `_WINSOCKAPI_` 매크로를 정의하면 깔끔하게 문제가 해결된다.

~~~cpp
#define _WINSOCKAPI_

#include <Windows.h>
#include <WinSock2.h>
~~~

개인적인 생각으로는 `WIN32_LEAN_AND_MEAN` 매크로를 정의하는 방법이 좋다고 생각한다. 불필요한 헤더를 줄이게 되고, 특정 헤더가 필요 할 때 추가하면 되니까.

# references

https://javawoo.tistory.com/42
http://egloos.zum.com/sweeper/v/1940473
https://docs.microsoft.com/en-us/windows/win32/winprog/using-the-windows-headers
https://m.blog.naver.com/PostView.nhn?blogId=sorkelf&logNo=40172871049&proxyReferer=https%3A%2F%2Fwww.google.com%2F
https://www.ikpil.com/596
https://code1018.tistory.com/198
