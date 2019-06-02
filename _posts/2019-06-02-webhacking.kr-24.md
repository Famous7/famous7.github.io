---
title: "webhacking.kr 24번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[24번 바로가기](http://webhacking.kr/challenge/bonus/bonus-4/)

24번 문제에 들어가면 아래와 같은 화면을 볼 수 있다. 접속한 IP와 User-Agent가 출력된다.

![24번_1](/img/24번_1.JPG)

소스코드를 보면 'index.phps'에 소스코드가 있다고 한다. 소스코드를 보자.

```php
extract($_SERVER);
extract($_COOKIE);

if(!$REMOTE_ADDR) $REMOTE_ADDR=$_SERVER[REMOTE_ADDR];

$ip=$REMOTE_ADDR;
$agent=$HTTP_USER_AGENT;


if($_COOKIE[REMOTE_ADDR])
{
$ip=str_replace("12","",$ip);
$ip=str_replace("7.","",$ip);
$ip=str_replace("0.","",$ip);
}

echo("<table border=1><tr><td>client ip</td><td>$ip</td></tr><tr><td>agent</td><td>$agent</td></tr></table>");

if($ip=="127.0.0.1")
{
@solve();
}

else
{
echo("<p><hr><center>Wrong IP!</center><hr>");
}
```

`extract()` 함수를 사용해 `$_SERVER`, `$_COOKIE`에 있는 값을 변수화 시킨 뒤 접속한 IP가 `127.0.0.1`이면 문제가 풀리는 구조이다.
쿠키에 `REMOTE_ADDR`이라는 값이 있으면 그 값을 `$ip`에 저장하고 없으면 `$_SERVER[REMOTE_ADDR]`을 `$ip`에 넣는다.


`$_SERVER[REMOTE_ADDR]`는 접속하는 IP 헤더에 설정된 src 주소로 설정되기 때문에 localhost인 `127.0.0.1`로 설정할 수 없다.
따라서 쿠키의 `REMOTE_ADDR`필드에 `127.0.0.1`을 넣어야 한다.


문제는 밑에 있는 if문에서 '12', '7.', '0.'이 필터링되는 건데 `str_replace()`함수는 `str_replace("12", "", "1122")`를 넣으면 "1122" 중 가운데 있는 "12"만 변경하기 때문에
쉽게 우회할 수 있다.

`112277..00..00..1` 을 쿠키에디터 등을 사용해 쿠키의 `REMOTE_ADDR`에 설정하고 새로고침하면 문제가 클리어된다.

![24번_2](/img/24번_2.JPG)
