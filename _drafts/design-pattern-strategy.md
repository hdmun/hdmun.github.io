---
layout: post
title: "[design pattern] 전략 패턴(Strategy)"
description: >
  
tags: [devlog]
author: hdmun
comments: true
---

**전략** 패턴에 대해 알아보자.  


간단히 파이썬 코드로 짜보았다.  


~~~python
class Context(object):
    def __init__(self):
        pass

    def interface(self):
        pass


class IStrategy(object):
    def interface(self):
        raise NotImplementedError()

~~~
