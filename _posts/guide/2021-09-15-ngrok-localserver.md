---
layout: post
title: "ngrok을 사용하여 외부에서 로컬 호스트 접속 테스트하기"
description: >
  ngrok을 사용하여 외부에서 로컬 호스트로 접속을 가능하게 하는 방법을 설명합니다.
tags: [dev, guide, tool]
author: hdmun
comments: true
---

> https://blog.outsider.ne.kr/1159


로컬 호스트 서버를 외부에서 접속 가능하게 만들어 주는 툴이다. 

개발 환경에서 가끔 외부 접속 테스트를 할 경우가 생기는데, 별도로 세팅하려면 방화벽 설정 등 너무 손이 많이 간다.

사용해보니 CLI 명령어 하나로 손 쉽게 외부와 통신을 하게 해주어 간단하게 기록해 놓고자 한다. 

일단 https://ngrok.com 에 접속해서 가입을 하자.

가입을 하면 아래와 같은 페이지가 나오는데 사용하기 위해선 `ngrok` 바이너리를 다운 받아야한다. 자신이 사용하는 플랫폼에 맞게 다운받자. 


![image](https://user-images.githubusercontent.com/48820928/134678424-39a472f0-3514-4a59-9127-7bdfc280fd4b.png)


윈도우의 경우 다운로드 받은 zip 파일에 `ngrok.exe` 만 들어있다.

사용을 하기 위해서는 토큰 값이 필요하다.

로그인 첫 화면에 아래와 같이 `2. Connect your account` 영역에 `authtoken` 옆에 있는 값이 사용자의 토큰 값이다. 


![image](https://user-images.githubusercontent.com/48820928/134678563-c5d2fcee-3c53-4af4-91e2-83785bed13aa.png)


이 토큰을 사용할려면 `ngrok.yml` 파일에 세팅이 필요하다. 아래와 같이 실행 인자를 줘서 실행해보자.


    ngrok authtoken [YOURETOKEN]

실행하고 나면 윈도우 기준 `%USERPROFILE%\.ngrok2\ngrok.yml` 위치에 토큰 세팅 파일이 생성된다.

이제 세팅은 끝났다. 아래와 같이 실행해서 테스트를 해보자.


    // 3000은 포트 번호
    ngrok http 3000

아래는 개발 중인 슬랙 앱 테스트를 위해 실행한 화면이다. 통신 요청이 올 때마다 로그가 찍히게 된다. 

참고로 Free 플랜은 `ngrok` 을 실행할 때 마다 `url` 이 매번 바뀌게 된다.

![image](https://user-images.githubusercontent.com/48820928/134678613-c99d0ca5-2722-4554-b9ea-8d6831a3be77.png)

# 마무리

세팅, 사용법이 블로그에 글을 쓰기조차 민망할 정도로 간단하지만 단순 기록 용도로 한 번 정리해보았다. 

`rgrok help` 로 확인을 해보니 `tcp` 통신도 지원을 하는 것 같은데, 지금 당장은 필요하지 않아 `tcp` 소켓 서버 개발을 할 때 한 번 사용 방법, 주의 사항을 정리해봐야겠다.



# Reference

https://velog.io/@nawnoes/express%EC%97%90%EC%84%9C-ngrok%EC%9C%BC%EB%A1%9C-%EC%99%B8%EB%B6%80%EC%97%90-%EC%84%9C%EB%B2%84-%EA%B3%B5%EA%B0%9C%ED%95%98%EA%B8%B0


