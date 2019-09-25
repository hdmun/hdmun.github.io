---
layout: post
title: "깃랩 페이지(gitlab page) 세팅하기"
description: >
  
tags: [devlog]
author: hdmun
comments: true
---

지금 블로그는 `깃허브 페이지(Github Pages)`에서 `jekyll`을 사용하여 호스팅하고 있다.

`깃랩(Gitlab)`에서도 비슷한 서비스를 제공하길래 `jekyll` 세팅하는 방법을 정리해보았다.

# 설치

설치하는 방법은 `깃허브 페이지(Github Pages)`와 거의 동일하다.

다만 추가로 `CI` 설정을 해주어야 한다. 저장소 루트에 `.gitlab-ci.yml` 파일을 만들고 아래와 같이 저장하자.


~~~yml
# .gitlab-ci.yml
image: ruby:2.3

variables:
  JEKYLL_ENV: production
  LC_ALL: C.UTF-8

before_script:
  - bundle install

pages:
  stage: deploy
  script:
  - bundle exec jekyll build -d public
  artifacts:
    paths:
    - public
  only:
  - master
~~~


https://gitlab.com/pages/jekyll

# 참고할만한 것들

## Gitlab Pages
https://about.gitlab.com/product/pages/

## Gitlab Pages examples
https://gitlab.com/pages?page=1

`jekyll` 말고도 다른 예제들이 포함되어 있다.

## Gitlab CI
https://about.gitlab.com/product/continuous-integration/
