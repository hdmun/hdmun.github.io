---
layout: post
title: "[design pattern] 의존성 주입(Dependency Injection)"
description: >
  
tags: [devlog]
author: hdmun
comments: true
---

**의존성 주입** 패턴에 대해 알아보자.  


간단히 파이썬 코드로 짜보았다.  


~~~python
class CharacterData(object):
    def __init__(self):
        pass


class User(object):
    def __init__(self):
        self._character_data = CharacterData()
~~~


~~~python
class CharacterData(object):
    def __init__(self):
        pass


class User(object):
    def __init__(self, character_data):
        pass
~~~
