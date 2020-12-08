---
layout: post
title: "비주얼 스튜디오 경고 레벨 올리기 - warning level 4"
description:
tags: [cpp, visualstudio]
author: hdmun
comments: true
---

[#cpp #visual_studio 내가 warning level 4로 만들어 봐서 아는데 원본 글 링크](http://ohyecloudy.com/pnotes/archives/visual-studio-warning-level-4/)

현재 일하고 있는 프로젝트는 대부분 라이브러리 프로젝트는 warning level 2, 서비스 프로젝트는 warning level 1로 되어있다. (왜 그렇게 세팅했는지는 모른다)

예전부터 warning level을 올릴 생각을 하고 있었는데


한 번에 warning level 4로 올리는걸 시도했으나 예상보다 warning이 많이 발생해 2, 3, 4 순서로 올리면서 작업하기로 했다. (빌드 시간 압박도 한 몫했다)

제거한 경고와 제거하지 않은 경고를 나눠서 정리했고, 위 레퍼런스 글에 있는 경고는 제외했다.

# 작업 준비

원글과 거의 동일하게 작업 준비를 했다. 몇 가지 다르게 한 점은 다음과 같다.

## warning level을 단계적으로 올리면서 warning을 비활성화

서비스 중인 프로젝트이기 때문에 경고 레벨을 한 번에 level 4까지 올려
비활성화 작업을 하기엔 빌드 시간 때문에 작업 시간이 너무 오래걸려
아래와 같이 단계 적으로 비활성화 작업을 했다.

1. warning level 2로 올리고 발생하는 경고들 비활성화, 커밋
2. warning level 3으로 올리고 발생하는 경고들 비활성화, 커밋
3. warning level 4로 올리고 발생하는 경고들 비활성화, 커밋

##

## 제거한 경고들

### [C4099 - level 2](https://docs.microsoft.com/ko-kr/cpp/error-messages/compiler-warnings/compiler-warning-level-2-c4099?view=vs-2015)

`class`로 선언하고 `struct`로 구현하거나 그 반대인 경우 발생한다.

~~~cpp
class ItemData;
struct ItemData {};
~~~

여러 프로젝트에서 공유하는 코드에서 발생하고 있었다. `struct`로 선언/정의된 코드에 접근제한자 `public`을 추가하여 `class`로 통일하는 방향으로 해결하였다.

## 제거하지 않은 경고들
