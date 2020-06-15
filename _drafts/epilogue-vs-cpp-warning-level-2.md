---
layout: post
title: "비주얼 스튜디오 경고 레벨 올리기 - warning level 2"
description:
tags: [cpp, visualstudio]
author: hdmun
comments: true
---

[#cpp #visual_studio 내가 warning level 4로 만들어 봐서 아는데 원본 글 링크](http://ohyecloudy.com/pnotes/archives/visual-studio-warning-level-4/)

위 글을 읽고 삘받아서 작업을 해보았다.

현재 몸 담고 있는 프로젝트는 대부분 경고 레벨이 1로 되어있다. 레벨 4로 올려서 정리해보니 warning 종류가 63개나 되었다. 부담스러운 숫자다.

라이브 서비스다 보니 이슈 발생 빈도를 줄이기 위해 점진적으로 작업을 할 필요성을 느꼈다.

한 번에 경고 레벨 4로 올리기엔 부담스러워 2, 3, 4 순서로 올리면서 작업하기로 하였다. 경고 레벨별로 제거 했었던 경고들을 정리한다. 위 레퍼런스 글에 있는 경고는 제외하였다.

## warning level 2

### [C4305 'type1'에서 'type2' (으)로 잘림](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-warnings/compiler-warning-level-1-c4305?view=vs-2019)

~~~cpp
~~~
