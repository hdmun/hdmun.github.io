---
layout: post
title: "python str 문자열에서 datetime UTC 시간대로 변경하기"
subtitle: 파이썬 str 문자열에서 datetime UTC 시간대로 변환하는 방법에 대해 설명한다.
categories: [python]
tags: [utc, python]
author: hdmun
comments: true
---

웹이나 오픈 API에서 데이터 수집을 하다보면 한국 시간대의 문자열로 `datetime` 값을 수집을 하게 된다. 그리고 수집한 데이터를 `datetime` 객체로 변환이 필요할 때마다 `datetime.strptime` 함수를 사용하게 된다.

이러한 상황에서 UTC 시간대로 변경이 필요할 경우 시간대 설정 및 `datetime` 값을 변환하는 방법에 대해 알아보자.


## 문자열을 `datetime` 객체로 바꾼 후 시간대를 포함해서 출력

### code
~~~py
import datetime

date_string = '2019-09-25 09:00:00.000'
format_ = '%Y-%m-%d %H:%M:%S.%f'
dt_strptime = datetime.datetime.strptime(date_string, format_)

print(dt_strptime.strftime('%Y-%m-%d %H:%M:%S %Z %z'))
~~~

### output
~~~
2019-09-25 09:00:00
~~~

`datetime.strptime` 함수를 사용해서 `datetime` 객체로 변환했을 때 시간대 설정이 되어있지 않다. 시간대를 설정해주자.


## 변환된 `datetime`에 시간대 추가

`replace` 함수를 사용해서 시간대를 추가해주자. 한국 시간대(KST)는 UTC 기준 +09:00 이므로 9시간을 더해준다.

하드코딩이 싫은 경우 로컬 타임과 UTC 시간의 차이를 계산해서 구해줘도 된다.

### code
~~~py
import datetime

date_string = '2019-09-25 09:00:00.000'
format_ = '%Y-%m-%d %H:%M:%S.%f'
dt_strptime = datetime.datetime.strptime(date_string, format_)

print(dt_strptime.strftime('%Y-%m-%d %H:%M:%S %Z %z'))


timedelta_ = datetime.datetime.now() - datetime.datetime.utcnow()
utc_kst_hours = timedelta_.seconds // 3600

timezon_kst = datetime.timezone(datetime.timedelta(hours=utc_kst_hours))
dt_timezone = dt_strptime.replace(tzinfo=timezon_kst)

print(dt_timezone.strftime('%Y-%m-%d %H:%M:%S %Z %z'))
~~~

### output
~~~
2019-09-25 09:00:00
2019-09-25 09:00:00 UTC+09:00 +0900
~~~

UTC 기준 +09:00 시간이라는 시간대 정보가 추가된 것을 볼 수 있다. 하지만 `replace` 함수는 `datetime` 객체에 시간대 값만 추가가 된거라 실제 날짜, 시간 값을 변환시켜주지 않는다.

객체의 날짜, 시간 값을 시간대에 맞춰 UTC 시간대로 변경해주자.


## UTC 시간대로 실제 날짜, 시간 값 변경하기

`astimezone` 함수를 사용해 실제 날짜, 시간 값을 KST 기준에서 UTC 기준으로 변경해주자

### code
~~~py
import datetime

date_string = '2019-09-25 09:00:00.000'
format_ = '%Y-%m-%d %H:%M:%S.%f'
dt_strptime = datetime.datetime.strptime(date_string, format_)

print(dt_strptime.strftime('%Y-%m-%d %H:%M:%S %Z %z'))


timedelta_ = datetime.datetime.now() - datetime.datetime.utcnow()
utc_kst_hours = timedelta_.seconds // 3600

timezon_kst = datetime.timezone(datetime.timedelta(hours=utc_kst_hours))
dt_timezone = dt_strptime.replace(tzinfo=timezon_kst)

print(dt_timezone.strftime('%Y-%m-%d %H:%M:%S %Z %z'))


dt_timezone = dt_timezone.astimezone(datetime.timezone.utc)

print(dt_timezone.strftime('%Y-%m-%d %H:%M:%S %Z %z'))
~~~

### output
~~~
2019-09-25 09:00:00
2019-09-25 09:00:00 UTC+09:00 +0900
2019-09-25 00:00:00 UTC +0000
~~~

UTC 시간대로 변경된 것을 볼 수 있다. 다만 이 방식에는 단점이 있는데 지역 시간대가 특정 시점을 기준으로 바뀔 수 있다는 점이다. 그래서 위 코드를 기준으로 하게 되면 현재 KST  시간을 기준으로 UTC 타임을 계산하므로 바뀌고 난 후의 시점이나 바뀌기 전 시점의 `datetime`을 변환할 때는 문제가 생긴다.

이유와 예시는 아래 스포카 기술 블로그에 자세히 설명되어있다. 읽어보자.

> 파이썬의 시간대에 대해 알아보기(datetime.timezone) - 스포카 기술 블로그  
 https://spoqa.github.io/2019/02/15/python-timezone.html


따라서 변경 이력이 포함되어 있는 `pytz` 패키지를 사용하면 위와 같은 문제는 없어진다. `pytz`로 KST에서 UTC 시간대로 변경하는 방법을 알아보자.

## pytz 사용하기

### 패키지 설치
~~~cmd
pip install pytz
~~~

### code
~~~py
import datetime
import pytz

def convert_utc(date_string):
    dt_strptime = datetime.datetime.strptime(date_string, '%Y-%m-%d %H:%M:%S.%f')
    # 시간대 설정
    tz_kst = pytz.timezone('Asia/Seoul')
    dt_kst = tz_kst.localize(dt_strptime)
    print(dt_kst.strftime('%Y-%m-%d %H:%M:%S %Z %z'))

    # 시간, 날짜 값 변경하기
    dt_utc_from_kst = dt_kst.astimezone(pytz.utc)
    print(dt_utc_from_kst.strftime('%Y-%m-%d %H:%M:%S %Z %z'))
    print('')


dt_strptime = convert_utc('2019-09-25 09:00:00.000')
dt_strptime = convert_utc('1987-09-25 09:00:00.000')
dt_strptime = convert_utc('1960-09-25 09:00:00.000')
~~~

### output
~~~
2019-09-25 09:00:00 KST +0900
2019-09-25 00:00:00 UTC +0000

1987-09-25 09:00:00 KDT +1000
1987-09-24 23:00:00 UTC +0000

1960-09-25 09:00:00 KST +0830
1960-09-25 00:30:00 UTC +0000
~~~

확인 해보면 특정 시점에 따라 변경되는 시간 값이 다르다.


# 결론

`pytz` 패키지를 써서 UTC로 변환해야겠다.


# 참고한 글 (references)

아래 링크는 이 글을 `timezone` 변경 방법을 찾으면서 참고한 글이다.

파이썬 `datetime.timezone`과 `pytz` 에 대해 자세하게 설명되어있다. 한 번 읽어보는 것을 추천한다.

## 파이썬의 시간대에 대해 알아보기(datetime.timezone) - 스포카 기술 블로그  
https://spoqa.github.io/2019/02/15/python-timezone.html

## Python, datetime timezone 삽질 정리
http://abh0518.net/tok/?p=635

## pytz 패키지 문서
http://pytz.sourceforge.net/

## pytz – 세계 시간대 정의를 위한 Python 라이브러리
https://edykim.com/ko/post/pytz-python-library-for-world-time-zone-definition/

## [Python] datetime timezone 변경방법
https://brownbears.tistory.com/356
