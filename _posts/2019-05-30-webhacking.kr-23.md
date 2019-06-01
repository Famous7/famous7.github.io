---
title: "webhacking.kr 23번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[23번 바로가기](http://webhacking.kr/challenge/bonus/bonus-3/)

23번 문제에 들어가면 아래와 같은 화면이 나온다. `<script>alert(1);</script> `를 삽입하라는 걸로 봐선
XSS 문제 같다. 페이지 소스를 봐도 별다른게 보이지 않는다. 200점짜리 쉬운 문제니까 우선 값을 넣어보자.

![23번_1](/img/23번_1.JPG)

'a'를 입력하면 화면에 'a'가 삽입된다. 'aa'를 넣으면 필터링된다.

![23번_2](/img/23번_2.JPG)

숫자를 넣어보니 숫자는 길이에 관계 없이 잘나온다.

![23번_3](/img/23번_3.JPG)

![23번_4](/img/23번_4.JPG)

특수문자도 잘 나오는걸로 봐서는 일반적인 문자만 길이가 2 이상이 되면 필터링되는 것 같다.

`<p>a</p>`를 넣어보면 p태그가 정상적으로 동작한다. 페이지 소스코드를 보면 html 태그가 잘 삽입되어 있다.

![23번_5](/img/23번_5.JPG)

html 태그는 잘 삽입되니까 문자열 길이 필터링만 우회하면 된다. `< s c r i p t >a l e r t ...` 이런식으로 넣으면 잘 들어가지만, 공백 때문에 스크립트로 인식되지 않는다.

보통 php에서 POSIX Regex 함수 중 하나인 `eregi()`로 이런 필터링을 많이 한다. 근데 이 함수는 `NULL`을 만나면 정규식 검사를 멈추는 취약점이 있다. 이건 PHP 5.3+ 이상 버전부터 나타난다. 예를 들어서 아래와 같은 로직이 있을 때 get 인자로 `%00hello`를 주면 필터링을 우회할 수 있다.


```php
$_cmd = $_GET[cmd];

if(eregi("hello", $_cmd))
  echo "Filtterd!!!"."<br>";
else
  echo $_cmd."<br>";

```

이런 POSIX Regex 함수들은 아래 함수들로 대체해야한다.

POSIX | PCRE
--------- | ---------
` ereg_replace()` | `preg_replace()`
`ereg()` | `preg_match()`
`eregi_replace()` | `preg_replace()`
`eregi()` | `preg_match()`
`split()` |  `preg_split()`
`spliti()` |  `preg_split()`
`sql_regcase()` |  No equivalent


이런 `eregi()`의 취약점을 이용해 앞에 `NULL`문자를 삽입한 `%00<script>alert(1);</script>`를 URL창에 입력하면 문제를 풀 수 있다.
이때 입력 폼을 통해 입력하면 `%00`에서 '%'가 `%25`로 인코딩되어서 들어가기 때문에 `%2500`이 되어버린다.

![23번_5](/img/23번_6.JPG)
